# Pulumi

Infrastructure as Code，把运维操作过程用代码来控制。

入门教程，提供了 aws/azure/gcp/k8s 的例子，对于来说，[k8s 的例子](https://www.pulumi.com/docs/get-started/kubernetes/)比较好理解一些，先用这个学习。

运行这个例子前，先要准备一个 k8s 集群，可以用 minikube 或 kind 在本地搭一个。

```
$ minikube start
$ kubectl config use-context minikube
```

安装 pulumi，略。

初次使用 pulumi 时，需要登录，因此需要注册 pulumi 账号。但我们只是需要在本地学习测试一下，可以用 `pulumi login --local` 在本地模拟，无须 pulumi 账号。

为了不在代码中暴露敏感信息，比如用户密码，token 等信息，我们可以把这个信息加密保存在 pulumi 中，一些常规的配置也可以以明文的方式保存在 pulumi 中。

加密需要设置一个 secret，放在 `PULUMI_CONFIG_PASSPHRASE` 环境变量中，比如：

```
export PULUMI_CONFIG_PASSPHRASE=a123456
```

保存明文配置：

```
$ pulumi config set isMinikube true
$ pulumi config get isMinikube
```

在代码中可以用 `requireBoolean()` 获取其值：

```ts
import * as pulumi from '@pulumi/pulumi'

const config = new pulumi.Config()
const isMinikube = config.requireBoolean('isMinikube')
```

保存加密配置：

```
$ pulumi config set myAuth xxxxx --secret
```

在代码中可以用 `getSecret()` 获取其值。(`requireSecret()`?)

```ts
import * as pulumi from '@pulumi/pulumi'

const config = new pulumi.Config()
const isMinikube = config.getSecret('myAuth')
```

这些值保存在 yaml 文件中，假如 stack 取名是 quickstart-dev，则保存在 Pulumi.quickstart-dev.yaml 中。打开 Pulumi.quickstart-dev.yaml 看一下内容：

```yaml
encryptionsalt: v1:NkT0DSGSSVY=:v1:PxvB6YqKVpNpdwZS:1/LAEeD2wOlVDe1fskirjVPLMcFQkQ==
config:
  quickstart:myAuth:
    secure: v1:p1njxLpO+tOOwsLU:vl/2ulT/fZ6mCIkhkWIXz4Dt+Ww=
  quickstart:isMinikube: "true"
```

看 index.ts 示例代码：

```ts
import * as k8s from '@pulumi/kubernetes'

const appLabels = { app: 'nginx' }
const deployment = new k8s.apps.v1.Deployment('nginx', {
  spec: {
    selector: { matchLabels: appLabels },
    replicas: 1,
    template: {
      metadata: { labels: appLabels },
      spec: { containers: [{ name: 'nginx', image: 'nginx' }] },
    },
  },
})

export const name = deployment.metadata.name
```

这个示例代码的作用就是动态地生成一个 deployment 的 yaml 配置文件，然后 pulumi 就会用这个 yaml 去进行真正的部署。

yaml 成了中间层。感觉有点像用 react 用 virtual dom 去生成 html。

`pulumi up` 执行操作，`pulumi destroy` 销毁操作。和 vagrant 的命令相似。
