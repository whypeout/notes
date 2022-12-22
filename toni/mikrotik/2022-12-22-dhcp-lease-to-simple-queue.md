# Membuat Simple Queue otomatis dari DHCP Lease client

sources:
- [https://gist.github.com/ibnux/59dd7d64bd9cb7e1e670b6ddd70ad991](https://gist.github.com/ibnux/59dd7d64bd9cb7e1e670b6ddd70ad991)
- 

```
:local queueName "Client-$leaseActMAC";
 
:local ipAdd "$leaseActIP/32"; 

:if ([:len [/queue simple find name=$queueName]] = 0) do={
:log info "No Queue";
        /queue simple add name=$queueName target=($ipAdd) limit-at=10M/4M max-limit=10M/4M comment=[/ip dhcp-server lease get [find where active-mac-address=$leaseActMAC && active-address=$leaseActIP] host-name];
    } else={
:log info "exists";
:local ada [/queue simple get [find name=$queueName] target];
        :log info "existing $ada";
        :if ($ada = $ipAdd) do={
            :log info "IP same $ada";
        } else={
            /queue simple set target=($ipAdd) [find name=$queueName];
        }
}
```

![image](https://user-images.githubusercontent.com/89820226/209099160-d0cc3843-db09-4cf4-a02f-eaebde3f5895.png)
![image](https://user-images.githubusercontent.com/89820226/209099216-868024bb-ec81-4d8b-8aa7-967d110cd154.png)
![image](https://user-images.githubusercontent.com/89820226/209099287-d26b8e6d-2e86-4257-8b60-d552bd54bec2.png)


![image](https://user-images.githubusercontent.com/89820226/209098999-0b462fa8-e0ea-494b-9473-8bb97aaa147c.png)


