### 评审记录

#### 第1个变更点：GitService.java

**变更文件**: ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/git/GitService.java
**变更类型**: MODIFY
**变更diff**:

```java
diff --git a/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/git/GitService.java b/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/git/GitService.java
index 37647e4..3c556bb 100644
--- a/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/git/GitService.java
+++ b/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/git/GitService.java
@@ -1,6 +1,7 @@
 package cn.ant0n.middleware.sdk.infrastructure.git;
 
 import org.apache.commons.lang.RandomStringUtils;
+import org.apache.commons.lang.StringUtils;
 import org.eclipse.jgit.api.Git;
 import org.eclipse.jgit.api.errors.GitAPIException;
 import org.eclipse.jgit.diff.DiffEntry;
@@ -42,14 +43,25 @@
         List<String> diffs = new ArrayList<>();
         // 获取项目根目录路径
         String projectPath = Paths.get(".").toAbsolutePath().normalize().toString();
+
         try (Repository repository = new FileRepositoryBuilder()
                 .setGitDir(new File(projectPath + "/.git"))
                 .build()) {
             Git git = new Git(repository);
+            // 获取指定分支的最新提交
+            ObjectId branchId = repository.resolve(this.branch + "^{commit}");
+            if(null == branchId) return null;
 
             RevWalk revWalk = new RevWalk(repository);
-            RevCommit lastCommit = revWalk.parseCommit(repository.resolve("HEAD"));
-            RevCommit parentCommit = revWalk.parseCommit(lastCommit.getParent(0).getId());
+            RevCommit lastCommit = revWalk.parseCommit(branchId);
+            RevCommit parentCommit = null;
+            // 检查是否有父提交
+            if (lastCommit.getParentCount() > 0) {
+                parentCommit = revWalk.parseCommit(lastCommit.getParent(0).getId());
+            } else {
+                System.out.println("no parent commit.");
+                return null;
+            }
 
             // 获取两个提交的tree对象
             ObjectId headTreeId = lastCommit.getTree().getId();
@@ -93,6 +104,7 @@
 
     @Override
     public String storeCodeView(String codeView) throws Exception {
+        if(StringUtils.isBlank(codeView)) return null;
         Git git = Git.cloneRepository()
                 .setURI(logRepoURI + ".git")
                 .setDirectory(new File("repo"))
```

**评审**:
- 添加了对 `StringUtils.isBlank()` 的调用，这是一个好的实践，因为它有助于避免空字符串或空白字符串导致的潜在错误。
- 在获取父提交时添加了错误处理，确保在没有父提交时能够优雅地处理异常。

**改进方案**:
- 检查 `StringUtils.isBlank()` 的使用是否适用于所有可能为空的字符串参数。
- 考虑在类中添加日志记录，以便在 `return null` 或其他错误处理时提供更多信息。

#### 第2个变更点：ChatGLM.java

**变更文件**: ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/openai/impl/ChatGLM.java
**变更类型**: MODIFY
**变更diff**:

```java
diff --git a/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/openai/impl/ChatGLM.java b/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/openai/impl/ChatGLM.java
index 1db797c..2d87b5d 100644
--- a/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/openai/impl/ChatGLM.java
+++ b/ant0n-openai-code-review-sdk/src/main/java/cn/ant0n/middleware/sdk/infrastructure/openai/impl/ChatGLM.java
@@ -31,6 +31,7 @@
 
     @Override
     public String completions(List<String> diffs) throws IOException {
+        if(diffs == null || diffs.isEmpty()) return null;
         Retrofit openAiCompletionsRetrofit = RetrofitConfig.getOpenAiCompletionsRetrofit(BASE_URL);
         IChatGL