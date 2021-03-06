+++
title = "UI发布指南"
weight = 7
chapter = true
+++

## GPG设置

详情请参见[发布指南](/cn/contribute/release/)。

## 发布Apache Maven中央仓库

### 设置settings.xml文件

将以下模板添加到 `~/.m2/settings.xml`中，所有密码需要加密后再填入。
加密设置可参考[这里](http://maven.apache.org/guides/mini/guide-encryption.html)。

```xml
<settings>
  <servers>
    <server>
      <id>apache.snapshots.https</id>
      <username> <!-- APACHE LDAP 用户名 --> </username>
      <password> <!-- APACHE LDAP 加密后的密码 --> </password>
    </server>
    <server>
      <id>apache.releases.https</id>
      <username> <!-- APACHE LDAP 用户名 --> </username>
      <password> <!-- APACHE LDAP 加密后的密码 --> </password>
    </server>
  </servers>
</settings>
```

### 更新版本说明

```
https://github.com/apache/shardingsphere/blob/master/shardingsphere-ui/RELEASE-NOTES.md
```

### 创建发布分支

假设从github下载的ShardingSphere源代码在`~/shardingsphere/`目录；假设即将发布的版本为`${RELEASE.VERSION}`。
创建`${RELEASE.VERSION}-release-ui`分支，接下来的操作都在该分支进行。

```shell
## ${name}为源码所在分支，如：master，dev-4.x
git clone --branch ${name} https://github.com/apache/shardingsphere.git ~/shardingsphere
cd ~/shardingsphere/
git pull
git checkout -b ${RELEASE.VERSION}-release-ui
git push origin ${RELEASE.VERSION}-release-ui
```

### 发布预校验

```shell
cd ~/shardingsphere/shardingsphere-ui
mvn release:prepare -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -DdryRun=true -Dusername=${Github用户名}
```

-Prelease: 选择release的profile，这个profile会打包所有源码、jar文件以及sharding-proxy的可执行二进制包。

-DautoVersionSubmodules=true：作用是发布过程中版本号只需要输入一次，不必为每个子模块都输入一次。

-DdryRun=true：演练，即不产生版本号提交，不生成新的tag。

### 准备发布

首先清理发布预校验本地信息。

```shell
cd ~/shardingsphere/shardingsphere-ui
mvn release:clean
```

然后准备执行发布。

```shell
cd ~/shardingsphere/shardingsphere-ui
mvn release:prepare -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -DpushChanges=false -Dusername=${Github用户名}
```

和上一步演练的命令基本相同，去掉了-DdryRun=true参数。

-DpushChanges=false：不要将修改后的版本号和tag自动提交至Github。

将本地文件检查无误后，提交至github。

```shell
git push
git push origin --tags
```

### 部署发布

```shell
cd ~/shardingsphere/shardingsphere-ui
mvn release:perform -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -Dusername=${Github用户名}
```

## 发布Apache SVN仓库

### 检出shardingsphere发布目录

如无本地工作目录，则先创建本地工作目录。

```shell
mkdir -p ~/ss_svn/dev/
cd ~/ss_svn/dev/
```

创建完毕后，从Apache SVN检出shardingsphere发布目录。

```shell
svn --username=${APACHE LDAP 用户名} co https://dist.apache.org/repos/dist/dev/shardingsphere
cd ~/ss_svn/dev/shardingsphere
```

### 添加gpg公钥

仅第一次部署的账号需要添加，只要`KEYS`中包含已经部署过的账户的公钥即可。

```shell
gpg -a --export ${GPG用户名} >> KEYS
```

### 将待发布的内容添加至SVN目录

创建版本号目录。

```shell
mkdir -p ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
cd ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
```

将源码包和二进制包添加至SVN工作目录。

```shell
cp -f ~/shardingsphere/shardingsphere-ui/shardingsphere-ui-distribution/shardingsphere-ui-src-distribution/target/*.zip ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
cp -f ~/shardingsphere/shardingsphere-ui/shardingsphere-ui-distribution/shardingsphere-ui-src-distribution/target/*.zip.asc ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
cp -f ~/shardingsphere/shardingsphere-ui/shardingsphere-ui-distribution/shardingsphere-ui-bin-distribution/target/*.tar.gz ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
cp -f ~/shardingsphere/shardingsphere-ui/shardingsphere-ui-distribution/shardingsphere-ui-bin-distribution/target/*.tar.gz.asc ~/ss_svn/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}
```

### 生成文件签名

```shell
shasum -a 512 apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip >> apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip.sha512
shasum -b -a 512 apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz >> apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz.sha512
```

### 提交Apache SVN

```shell
cd ~/ss_svn/dev/shardingsphere/
svn add shardingsphere-ui-${RELEASE.VERSION}
svn --username=${APACHE LDAP 用户名} commit -m "release shardingsphere-ui-${RELEASE.VERSION}"
```
## 检查发布结果

### 检查sha512哈希

```shell
shasum -c apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip.sha512
shasum -c apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz.sha512
```

### 检查gpg签名

首先导入发布人公钥。从svn仓库导入KEYS到本地环境。（发布版本的人不需要再导入，帮助做验证的人需要导入，用户名填发版人的即可）

```shell
curl https://dist.apache.org/repos/dist/dev/shardingsphere/KEYS >> KEYS
gpg --import KEYS
gpg --edit-key "${发布人的gpg用户名}"
  > trust

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5

  > save
```

然后进行gpg签名检查。

```shell
gpg --verify apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip.asc apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip
gpg --verify apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz.asc apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz
```

### 检查发布文件内容

#### 对比源码包与Github上tag的内容差异

```
curl -Lo tag-shardingsphere-ui-${RELEASE.VERSION}.zip https://github.com/apache/shardingsphere/archive/shardingsphere-ui-${RELEASE.VERSION}.zip
unzip tag-shardingsphere-ui-${RELEASE.VERSION}.zip
unzip apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src.zip
diff -r apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-src/ shardingsphere-shardingsphere-ui-${RELEASE.VERSION}/shardingsphere-ui/
```

#### 检查源码包的文件内容

- 检查源码包是否包含由于包含不必要文件，致使tarball过于庞大
- 存在`LICENSE`和`NOTICE`文件
- `NOTICE`文件中的年份正确
- 只存在文本文件，不存在二进制文件
- 所有文件的开头都有ASF许可证
- 能够正确编译，单元测试可以通过 (./mvnw install)
- 检查是否有多余文件或文件夹，例如空文件夹等

#### 检查二进制包的文件内容

解压缩`apache-shardingsphere-${RELEASE.VERSION}-shardingsphere-ui-bin.tar.gz`
进行如下检查:

- 存在`LICENSE`和`NOTICE`文件
- `NOTICE`文件中的年份正确
- 所有文本文件开头都有ASF许可证
- 检查第三方依赖许可证：
  - 第三方依赖的许可证兼容
  - 所有第三方依赖的许可证都在`LICENSE`文件中声名
  - 依赖许可证的完整版全部在`license`目录
  - 如果依赖的是Apache许可证并且存在`NOTICE`文件，那么这些`NOTICE`文件也需要加入到版本的`NOTICE`文件中

## 发起投票

### 投票阶段

1. ShardingSphere社区投票，发起投票邮件到`dev@shardingsphere.apache.org`。PMC需要先按照文档检查版本的正确性，然后再进行投票。
经过至少72小时并统计到3个`+1 PMC member`票后，即可进入下一阶段的投票。

2. 宣布投票结果,发起投票结果邮件到`dev@shardingsphere.apache.org`。

### 投票模板

1. ShardingSphere社区投票模板

标题：

```
[VOTE] Release Apache ShardingSphere UI ${RELEASE.VERSION}
```

正文：

```
Hello ShardingSphere Community,

This is a call for vote to release Apache ShardingSphere UI version ${RELEASE.VERSION}

Release notes:
https://github.com/apache/shardingsphere/blob/master/shardingsphere-ui/RELEASE-NOTES.md

The release candidates:
https://dist.apache.org/repos/dist/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION}/

Git tag for the release:
https://github.com/apache/shardingsphere/tree/shardingsphere-ui-${RELEASE.VERSION}/

Release Commit ID:
https://github.com/apache/shardingsphere/commit/xxxxxxxxxxxxxxxxxxxxxxx

Keys to verify the Release Candidate:
https://dist.apache.org/repos/dist/dev/shardingsphere/KEYS

Look at here for how to verify this release candidate:
https://shardingsphere.apache.org/community/en/contribute/release_ui/

GPG user ID:
${YOUR.GPG.USER.ID}

The vote will be open for at least 72 hours or until necessary number of votes are reached.

Please vote accordingly:

[ ] +1 approve 

[ ] +0 no opinion
 
[ ] -1 disapprove with the reason

PMC vote is +1 binding, all others is +1 non-binding.

Checklist for reference:

[ ] Download links are valid.

[ ] Checksums and PGP signatures are valid.

[ ] Source code distributions have correct names matching the current release.

[ ] LICENSE and NOTICE files are correct for each ShardingSphere repo.

[ ] All files have license headers if necessary.

[ ] No compiled archives bundled in source archive.
```

2. 宣布投票结果模板：

正文：

```
The vote to release Apache ShardingSphere UI ${RELEASE.VERSION} has passed.

7 PMC member +1 binding votes:

xxx
xxx
xxx
xxx
xxx
xxx
xxx

1 community +1 non-binding vote:
xxx

Thank you everyone for taking the time to review the release and help us. 
```

3. 宣布投票结果模板：

标题：

```
[RESULT][VOTE] Release Apache ShardingSphere UI ${RELEASE.VERSION}
```

正文：

```
We’ve received 3 +1 binding votes and one +1 non-binding vote:

+1 binding, xxx
+1 binding, xxx
+1 binding, xxx

+1 non-binding, xxx

Thank you everyone for taking the time to review the release and help us. 
I will process to publish the release and send ANNOUNCE.
```

## 完成发布

### 将源码、二进制包以及KEYS从svn的dev目录移动到release目录

```shell
svn mv https://dist.apache.org/repos/dist/dev/shardingsphere/shardingsphere-ui-${RELEASE.VERSION} https://dist.apache.org/repos/dist/release/shardingsphere/ -m "transfer packages for shardingsphere-ui-${RELEASE.VERSION}"
svn delete https://dist.apache.org/repos/dist/release/shardingsphere/KEYS -m "delete KEYS"
svn cp https://dist.apache.org/repos/dist/dev/shardingsphere/KEYS https://dist.apache.org/repos/dist/release/shardingsphere/ -m "transfer KEYS for shardingsphere-ui-${RELEASE.VERSION}"
```

### 合并Github的release分支到`master`, 合并完成后删除release分支

```shell
git checkout master
git merge origin/${RELEASE.VERSION}-release-ui
git push
git push --delete origin ${RELEASE.VERSION}-release-ui
```

### 更新下载页面

https://shardingsphere.apache.org/document/current/en/downloads/

https://shardingsphere.apache.org/document/current/cn/downloads/

`最新版本`中保留两个最新的版本。Incubator阶段历史版本会自动归档到[Archive repository](https://archive.apache.org/dist/incubator/shardingsphere/)

### GitHub版本发布

在[GitHub Releases](https://github.com/apache/shardingsphere/releases)页面的`shardingsphere-ui-${RELEASE_VERSION}`版本上点击`Edit`

编辑版本号及版本说明，并点击`Publish release`

### 发送邮件到`dev@shardingsphere.apache.org`和`announce@apache.org`通知完成版本发布

通知邮件模板：

标题：

```
[ANNOUNCE] Apache ShardingSphere UI ${RELEASE.VERSION} available
```

正文：

```
Hi all,

Apache ShardingSphere Team is glad to announce the new release of Apache ShardingSphere UI ${RELEASE.VERSION}.

ShardingSphere is an open-source ecosystem consisted of a set of distributed database middleware solutions, including 2 independent products, Sharding-JDBC & Sharding-Proxy. 
They both provide functions of data sharding, distributed transaction and database orchestration, applicable in a variety of situations such as Java isomorphism, heterogeneous language. 
Aiming at reasonably making full use of the computation and storage capacity of the database in a distributed system, ShardingSphere defines itself as a middleware, rather than a totally new type of database. 
As the cornerstone of many enterprises, relational database still takes a huge market share. 
Therefore, at the current stage, we prefer to focus on its increment instead of a total overturn.

Download Links: https://shardingsphere.apache.org/document/current/en/downloads/

Release Notes: https://github.com/apache/shardingsphere/blob/master/shardingsphere-ui/RELEASE-NOTES.md

Website: https://shardingsphere.apache.org/

ShardingSphere Resources:
- Issue: https://github.com/apache/shardingsphere/issues/
- Mailing list: dev@shardingsphere.apache.org
- Documents: https://shardingsphere.apache.org/document/current/



- Apache ShardingSphere Team

```
