# ElasticSearch

## filebeat

send logs from k8s to elastic cloud.

// TODO
// see PR for tidbcloud/infra.

关键的配置：

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: { { .Release.Namespace } }
  labels:
    app: filebeat
data:
  filebeat.yml: |-
    filebeat.inputs:
    - type: container
      paths:
      - /var/log/containers/*.log
      processors: # 仅对当前 input 生效的 processor
      - add_kubernetes_metadata: # 添加 k8s metadata
          host: ${NODE_NAME}
          matchers:
          - logs_path:
              logs_path: "/var/log/containers/"

    processors: # 全局 processor
    - include_fields:
        fields: ["host.name", "stream", "message", "kubernetes.namespace", "kubernetes.container.name", "kubernetes.pod.name"]
    # - drop_fields:
    #     fields: ["agent.ephemeral_id", "agent.hostname", "agent.id", "agent.type", "agent.version", "ecs.version", "input.type", "log.offset"]
    #     ignore_missing: true
    - rename:
        fields:
        - from: "host.name"
          to: "host_name"
        - from: "kubernetes.namespace"
          to: "namespace"
        - from: "kubernetes.container.name"
          to: "container_name" # use "container" will report error，因为 container field 已存在且为 object
        - from: "kubernetes.pod.name"
          to: "instance"
        ignore_missing: true
        fail_on_error: false

    cloud.id: observability-deployment:dXMtd2VzdC0yLmF3cy5mb3VuZC5pbyQ5ZTczY2VlNTlmMmU0M2ZiOTNiMjVhOGM2M2Q0Mjk4OSRlOTRiMDRlZjQ1MDk0NDMwOTc4ZTliZmM3ODhiNGI5Mw==
    cloud.auth: {{ .Values.config.esCloudAuth }}
```

filebeat 移除 fields 时，如果用下面的配置，会移除 kubernets 及所有的子 fileds 吗？(待测试)

```yaml
processors: # 全局 processor
  #     - include_fields:
  #         fields: ["host.name", "stream", "message", "kubernetes.namespace", "kubernetes.container.name", "kubernetes.pod.name"]
  - drop_fields:
      fields: ['kubernetes']
      ignore_missing: true
```

## kibana setup auth0 login

首先配置 auth0，跟着这篇文章的步骤：https://auth0.com/docs/protocols/saml-protocol/configure-auth0-as-saml-identity-provider?#manually-configure-sso-integrations

Application Type 随便选都行，这个不会影响后面的配置。

Allowed Callback URLs 填入下面配置中的 sp.acs。

elastic cloud 这边的配置

参考文档：https://www.elastic.co/guide/en/cloud/current/ec-securing-clusters-SAML.html

elastic:

```yaml
xpack:
  security:
    authc:
      realms:
        saml:
          soc-auth:
            order: 2
            attributes.principal: 'http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress'
            attributes.groups: 'roles'
            idp.metadata.path: 'https://tidb-soc2.us.auth0.com/samlp/metadata/xB3EUrAe2exEX1FK4ZRTLjrxRKGybUNA'
            idp.entity_id: 'urn:tidb-soc2.us.auth0.com'
            sp.entity_id: 'https://observability-deployment-f12c4c.kb.us-west-2.aws.found.io:9243/'
            sp.acs: 'https://observability-deployment-f12c4c.kb.us-west-2.aws.found.io:9243/api/security/v1/saml'
            sp.logout: 'https://observability-deployment-f12c4c.kb.us-west-2.aws.found.io:9243/logout'
```

attributes.principal 字段表示我应该用从 auth0 得到的用户信息的哪个字段来作为区分用户的标志。上面的配置表示使用用户的邮箱地址作为区分用户的标志。

kibana:

```yaml
xpack.security.authc.providers:
  saml.saml1:
    order: 0
    realm: soc-auth
    description: 'Log in with Auth0'
```
