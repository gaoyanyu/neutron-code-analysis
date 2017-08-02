# Neutron Ml2 Mechanism Driver 之 linuxbridge

*neutron/ml2/drivers/linuxbridge/mech_driver/mech_linuxbridge.py*

## `class LinuxbridgeMechanismDriver(mech_agent.SimpleAgentMechanismDriverBase)`

```
    supported_qos_rule_types = [qos_consts.RULE_TYPE_BANDWIDTH_LIMIT]

    def __init__(self):
        sg_enabled = securitygroups_rpc.is_firewall_enabled()
        super(LinuxbridgeMechanismDriver, self).__init__(
            constants.AGENT_TYPE_LINUXBRIDGE,
            portbindings.VIF_TYPE_BRIDGE,
            {portbindings.CAP_PORT_FILTER: sg_enabled})
```

### `def check_vlan_transparency(self, context)`

```
    def check_vlan_transparency(self, context):
        """Linuxbridge driver vlan transparency support."""
        return True
```
