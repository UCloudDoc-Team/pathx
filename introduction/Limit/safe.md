# 安全合规限制

- 购买加速区域在中国大陆地区的加速线路，如果您需要将业务域名解析到PathX的加速域名或加速IP上，按照中华人民共和国相关法律规定，该业务域名必须备案，没有备案的域名流量进入公有云，云厂商会执行封堵。
- UCloud免费为每个加速IP提供不超过 3 Gbps的基础攻击防御（不同地域支持的最大免费防护流量不同），当加速实例遭受DDoS攻击超过基础防护阈值后，UCloud会对加速区域入口IP采取封堵措施，加速实例将会做回源处理。若您的加速实例持续遭受DDoS攻击，产品方保留回收实例权利。
- 禁止加速域名直接提供HTTP/HTTPS访问。
