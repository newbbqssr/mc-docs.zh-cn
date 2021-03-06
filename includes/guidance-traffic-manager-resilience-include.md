## <a name="highly-available-solutions-with-azure-traffic-manager"></a>Azure 流量管理器的高可用解决方案
你需要确定是可以单独使用 Azure 流量管理器来满足工作负荷的高可用性要求，还是需要将流量管理器与其他 DNS 解决方案或流程组合使用。 你可以根据需要来使用：

* **单独的流量管理器**。 如果 99.99% 的运行时间可以满足你的工作负荷，则可单独使用流量管理器。 在流量管理器服务出现故障时，用户将无法访问你的工作负荷，直至重新建立流量管理器服务。
* **将其他流量管理器解决方案与 Azure 流量管理器配合使用**。 在流量管理器服务出现故障时，你可以更改 CNAME 记录，使之指向其他流量管理器服务。 仍然可以访问你的工作负荷，并且会将流量分布到承载工作负荷的所有位置。 这是最昂贵的解决方案，但对于需要更高 SLA 的工作负荷来说，这可能是必需的。