

pom.xml

<!--dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>-->


changelog.md

## How it works

Tunnel server/client use websocket protocol.

For example:

1. Arthas tunnel server listen at `192.168.1.10:7777`

1. Arthas tunnel client register to the tunnel server

    tunnel client connect to tunnel server with URL: `ws://192.168.1.10:7777/ws?method=agentRegister`

    tunnel server response a text frame message: `response:/?method=agentRegister&id=bvDOe8XbTM2pQWjF4cfw`

    This connection is `control connection`.

1. The browser try connect to remote arthas agent, start connect to tunnel server with URL: `'ws://192.168.1.10:7777/ws?method=connectArthas&id=bvDOe8XbTM2pQWjF4cfw`

1. Arthas server find the `control connection` with the id `bvDOe8XbTM2pQWjF4cfw`, then send a text frame to arthas client: `response:/?method=startTunnel&id=bvDOe8XbTM2pQWjF4cfw&clientConnectionId=AMku9EFz2gxeL2gedGOC`

1. Arhtas tunnel client open a new connection to tunnel server, URL: `ws://127.0.0.1:7777/ws/?method=openTunnel&clientConnectionId=AMku9EFz2gxeL2gedGOC&id=bvDOe8XbTM2pQWjF4cfw`. This connection is `tunnel connection`.

1. Arhtas tunnel client start connect to local arthas agent, URL: `ws://127.0.0.1:3658/ws`. This connection is `local connection`.

1. Forward websocket frame between `tunnel connection` and `local connection`.

```
browser <-> arthas tunnel server <-> arthas tunnel client <-> arthas agent
```


￼





package com.alibaba.arthas.tunnel.server.endpoint;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;

import com.alibaba.arthas.tunnel.server.TunnelServer;
import com.alibaba.arthas.tunnel.server.app.ArthasProperties;
import org.springframework.web.bind.annotation.CrossOrigin;

@Endpoint(id = "arthas")
@CrossOrigin(origins = "*")
public class ArthasEndpoint {

    @Autowired
    ArthasProperties arthasProperties;
    @Autowired
    TunnelServer tunnelServer;

    @ReadOperation
    public Map<String, Object> invoke() {
        Map<String, Object> result = new HashMap<String, Object>();

        result.put("version", this.getClass().getPackage().getImplementationVersion());
        result.put("properties", arthasProperties);

        result.put("agents", tunnelServer.getAgentInfoMap());
        result.put("clientConnections", tunnelServer.getClientConnectionInfoMap());
        result.put("code",200);
        result.put("message","success");
        return result;
    }

}






//package com.alibaba.arthas.tunnel.server.endpoint;
//
//import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
//import org.springframework.context.annotation.Configuration;
//import org.springframework.security.config.annotation.web.builders.HttpSecurity;
//import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
//
//@Configuration
//public class ActuatorSecurity extends WebSecurityConfigurerAdapter {
//    @Override
//    protected void configure(HttpSecurity httpSecurity) throws Exception {
//        httpSecurity.authorizeRequests().requestMatchers(EndpointRequest.toAnyEndpoint()).authenticated().anyRequest()
//                .permitAll().and().formLogin();
//    }
//}



