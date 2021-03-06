# Neutron Ipam 之 NeutronDbSubnet

将 subnet 数据库的记录转化为一个对象 `NeutronDbSubnet`，类似于 Neutron 的 object 一样。

*neutron/ipam/driver.py*

*neutron/ipam/drivers/neutrondb_ipam/driver.py*

## `class Subnet(object)`

抽象基类，定义了三个抽象方法：

### `def allocate(self, address_request)`

分配 Ip

### `def deallocate(self, address)`

回收 Ip

### `def get_details(self)`

获取该子网的详细信息

## `class NeutronDbSubnet(ipam_base.Subnet)`

### `def load(cls, neutron_subnet_id, ctx)`

类方法

1. 调用 `IpamSubnetManager.load_by_neutron_subnet_id` 根据子网 id 记载数据库 `IpamSubnet` 的一条记录
2. 根据 `IpamSubnet` 加载该子网的所有子网池（`IpamAllocationPool`）
3. 调用 `_fetch_subnet` 获取子网的数据库记录
4. 通过上述获得的消息，构造该类的一个实例，用来表示一个子网。

### `def _fetch_subnet(cls, context, id)`

类方法

通过调用 ml2 的 `_get_subnet` 获取子网的数据库记录

### `def __init__(self, internal_id, ctx, cidr=None, allocation_pools=None, gateway_ip=None, tenant_id=None, subnet_id=None)`

根据一条 subnet 数据库记录，创建一个 `NeutronDbSubnet` 实例。

### `def get_details(self)`

```
    def get_details(self):
        """Return subnet data as a SpecificSubnetRequest"""
        return ipam_req.SpecificSubnetRequest(
            self._tenant_id, self.subnet_manager.neutron_id,
            self._cidr, self._gateway_ip, self._pools)
```

### `def allocate(self, address_request)`

根据分配地址的请求，从子网中分配一个 Ip

1. 若请求是 `SpecificAddressRequest` 类型的，则调用 `_verify_ip` 验证这个被请求分配的 ip 地址是否合法
2. 对于其他类型的请求，则调用 `_generate_ip` 从该子网的地址池中获取一个 Ip
3. 调用 `IpamSubnetManager.create_allocation` 创建一个 `IpamAllocation` 数据库记录，表示刚才获取的地址以及被分配
4. 返回分配的 ip 地址

### `def _verify_ip(self, session, ip_address)`

1. 调用 `IpamSubnetManager.check_unique_allocation` 检查该子网的这个 Ip 地址是否已经被分配
2. 调用 `ipam_utils.check_subnet_ip` 检查该 Ip 地址相对于该子网的 cidr 来说是否可被分配

### `def _generate_ip(self, session, prefer_next=False)`

1. 调用 `IpamSubnetManager.list_allocations` 获取该子网已经分配的 Ip 地址的记录
2. 调用 `IpamSubnetManager.list_pools` 获取该子网的所有地址池
3. 将地址池中已分配的 Ip 地址去掉后，取出一个 ip 地址
4. 返回 Ip 地址，以及 Ip 地址对应的 ip 地址池的 id

### `def deallocate(self, address)`

1. 调用 `IpamSubnetManager.delet_allocation` 删除一个 `IpamAllocation` 数据库记录，表示地址被回收

### `def create_allocation_pools(cls, subnet_manager, session, pools, cidr)`

调用 `IpamSubnetManager.create_pool` 创建一个 `IpamAllocationPool` 数据库记录

### `def create_from_subnet_request(cls, subnet_request, ctx)`

根据请求创建一个子网。

1. 调用 `uuidutils.generate_uuid` 产生一个 `IpamSubnet` 数据库记录的 id
2. 构造一个 `IpamSubnetManager` 实例
3. 调用 `IpamSubnetManager.create` 创建一条 `IpamSubnet` 的数据库记录
4. 调用 `create_allocation_pools` 创建与 `IpamSubnet` 相关联的 `IpamAllocationPool` 数据库记录
5. 返回一个描述该 subne 的 `NeutronDbSubnet` 的实例。

### `def _no_pool_changes(self, session, pools)`

1. 获取当前子网所拥有的地址池
2. 对比待更新的地址池（pools）和当前子网的地址池是否一样，若一样则返回 True，否则返回 False

### `def update_allocation_pools(self, pools, cidr)`

更新子网的地址池。

1. 调用 `_no_pool_changes` 检查待更新的地址池是否与原地址池一致，若一致则直接退出
2. 调用 `IpamSubnetManager.delete_allocation_pools` 删除原地址池的数据库记录
3. 调用 `create_allocation_pools` 创建新的地址池数据库记录


















