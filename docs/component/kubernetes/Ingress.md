## Ingress是什么？

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#ingress-v1beta1-networking-k8s-io) 公开了从集群外部到集群内[服务](https://kubernetes.io/zh/docs/concepts/services-networking/service/)的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

下面是一个将所有流量都发送到同一 Service 的简单 Ingress 示例：

![](.\images\Ingress_structure.png)

可以将 Ingress 配置为服务提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及提供基于名称的虚拟主机等能力。 [Ingress 控制器](https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers) 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。 将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用 [Service.Type=NodePort](https://kubernetes.io/zh/docs/concepts/services-networking/service/#nodeport) 或 [Service.Type=LoadBalancer](https://kubernetes.io/zh/docs/concepts/services-networking/service/#loadbalancer) 类型的服务。



## Ingress控制器

Ingress需要Ingress控制器才能满足规则的要求，有控制器根据Ingress的规则来决定如何访问API。最常用的控制器包括Ingress-nginx，Ingress-HAProxy，Ingress-Istio等。

Ingress-nginx 为k8s维护的组件

Nginx-Ingress为Nginx维护的组件

## Ingress作用

控制在K8S的集群内，HTTP请求的路由转发，属于七层均衡器。也可以通过configmap方式做到四层代理

 **四层代理、session保持、定制配置、流量控制**

也意味着可以通过对URI的路径进行匹配不同的路由规则。

Ingress通过规则来定义，每个 HTTP 规则都包含以下信息：

- 可选的 `host`。在此示例中，未指定 `host`，因此该规则适用于通过指定 IP 地址的所有入站 HTTP 通信。 如果提供了 `host`（例如 foo.bar.com），则 `rules` 适用于该 `host`。
- 路径列表 paths（例如，`/testpath`）,每个路径都有一个由 `serviceName` 和 `servicePort` 定义的关联后端。 在负载均衡器将流量定向到引用的服务之前，主机和路径都必须匹配传入请求的内容。
- `backend`（后端）是 [Service 文档](https://kubernetes.io/zh/docs/concepts/services-networking/service/)中所述的服务和端口名称的组合。 与规则的 `host` 和 `path` 匹配的对 Ingress 的 HTTP（和 HTTPS ）请求将发送到列出的 `backend`。

## 局限性

暂时仅提供

负载均衡算法有限，仅支持Round Robin 和ewma均值， 因为在K8S认为中，服务里面的各个POD都应该是一模一样的，可根据不同的业务分类创建不同的服务。

不支持部分校验功能例如JWT，

多个服务有路径冲突，或者部分使用了绝对路径