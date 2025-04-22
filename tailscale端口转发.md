# Tailscale 配置端口转发文档

## 一、概述

本文档介绍了如何在 Tailscale 网络中配置端口转发，使得外部设备能够通过 A 机器的公网 IP 和特定端口访问 B 机器上的服务。适用于已经加入同一个 Tailscale 网络的 A 和 B 机器，其中 A 机器具有公网 IP，B 机器没有公网 IP 但运行着网页服务。

## 二、准备工作

1. **确保 A 和 B 已经加入同一个 Tailscale 网络**
   - 在 A 和 B 机器上安装并配置 Tailscale，确保两者能够相互通信。
   - 使用以下命令查看当前 Tailscale 网络中的设备信息，确认 B 已经成功组网：
     ```bash
     tailscale status
     ```

2. **确认 B 机器上的服务正在运行**
   - 在 B 机器上运行以下命令，确保服务正在监听特定的端口（例如 3000）：
     ```bash
     curl 127.0.0.1:3000
     ```

## 三、配置端口转发

### 3.1 在 A 机器上配置 IPTABLES 规则

1. **添加 PREROUTING 规则**
   - 将外部访问 A 的特定端口（例如 3002）的流量转发到 B 的服务端口（例如 3000）：
     ```bash
     sudo iptables -t nat -A PREROUTING -p tcp --dport 3002 -j DNAT --to-destination <B的Tailscale_IP>:3000
     ```
     - `<B的Tailscale_IP>`：替换为 B 机器在 Tailscale 网络中的 IP 地址。

2. **添加 POSTROUTING 规则**
   - 添加 MASQUERADE 规则，确保转发后的流量能够正确返回：
     ```bash
     sudo iptables -t nat -A POSTROUTING -p tcp -d <B的Tailscale_IP> --dport 3000 -j MASQUERADE
     ```

### 3.2 在 A 机器上配置 UFW

1. **开放指定端口**
   - 确保 A 机器的防火墙允许外部访问指定端口（例如 3002）：
     ```bash
     sudo ufw allow 3002
     ```

2. **验证 UFW 配置**
   - 查看当前 UFW 配置，确认端口已开放：
     ```bash
     sudo ufw status
     ```

## 四、验证配置

1. **在 A 机器上测试**
   - 在 A 机器上运行以下命令，验证是否能够通过本地端口访问 B 的服务：
     ```bash
     curl 127.0.0.1:3002
     ```

2. **在外部设备（C 机器）上测试**
   - 在 C 机器上运行以下命令，验证是否能够通过 A 的公网 IP 和指定端口访问 B 的服务：
     ```bash
     curl http://<A的公网IP>:3002
     ```

## 五、注意事项

- **确保 Tailscale 网络正常工作**
  - 在 A 机器上运行 `tailscale status`，确认 B 机器是否在线。
- **检查 B 机器的防火墙配置**
  - 确保 B 机器的防火墙允许来自 A 机器的流量。
- **持久化 IPTABLES 规则**
  - 如果需要重启 A 机器后保持规则有效，可以使用以下命令保存规则：
    ```bash
    sudo iptables-save > /etc/iptables/rules.v4
    ```

## 六、常见问题及解决方法

| 问题 | 解决方法 |
| --- | --- |
| `curl: (7) Failed to connect` | 检查 `PREROUTING` 和 `POSTROUTING` 规则是否正确配置，确保 `MASQUERADE` 规则存在。 |
| `Could not delete non-existent rule` | 使用 `sudo iptables -t nat -L -n -v` 查看规则编号，使用编号删除规则。 |
| 服务无法访问 | 确认 B 机器上的服务是否正在运行，检查 Tailscale 网络连接是否正常。 |

## 七、参考文档

- [Tailscale 官方文档](https://tailscale.com/kb/)
- [IPTABLES 官方文档](https://www.netfilter.org/documentation.html)

希望这份文档能够帮助你成功配置 Tailscale 端口转发。如有任何问题，请参考上述文档或联系技术支持。