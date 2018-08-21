# Istio完整配置文件

本文写的时候版本`v1.0.0`，

写这个文档的目的是[官方文档][1]缺少一份完整的配置例子，在写配置文件的时候很不方便需要反复查阅。

## Gateway

<pre>
apiVersion: networking.istio.io/v1alpha3
kind: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway">Gateway</a>
metadata:
  name: my-gateway
spec:
  selector: map&lt;string, string&gt;
  servers: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Server">Server[]</a>
  - port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Port">Port</a>
      number: uint32
      name: string
      protocol: string
    hosts: string[]
    tls: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Server-TLSOptions">Server.TLSOptions</a>
      httpsRedirect: bool
      mode: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Server-TLSOptions-TLSmode">Server.TLSOptions.TLSmode</a>
      serverCertificate: string
      privateKey: string
      caCertificates: string
      subjectAltNames: string[]
</pre>

## VirtualService

<pre>
apiVersion: networking.istio.io/v1alpha3
kind: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService">VirtualService</a>
metadata:
  name: reviews-route
spec:
  hosts: string[]
  gateways: string[]
  http: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPRoute">HTTPRoute[]</a>
  - match: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest">HTTPMatchRequest[]</a>
    - uri: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#StringMatch">StringMatch</a>
        exact: string (oneof)
        prefix: string (oneof)
        regex: string (oneof)
      scheme: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#StringMatch">StringMatch</a>
      method: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#StringMatch">StringMatch</a>
      authority: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#StringMatch">StringMatch</a>
      headers: map&lt;string, <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#StringMatch">StringMatch</a>&gt;
      port: uint32	
      sourceLabels: map&lt;string, string&gt;
      gateways: string[]
    route: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#DestinationWeight">DestinationWeight[]</a>
    - destination: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Destination">Destination</a>
        host: string
        subset: string
        port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#PortSelector">PortSelector</a>
          number: uint32
      weight: int32
    redirect: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPRedirect">HTTPRedirect</a>
      uri: string
      authority: string
    rewrite: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPRewrite">HTTPRewrite</a>
      uri: string
      authority: string    
    timeout: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
    retries: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPRetry">HTTPRetry</a>
      attempts: int32
      perTryTimeout: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
    fault: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPFaultInjection">HTTPFaultInjection</a>
      delay: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPFaultInjection-Delay">HTTPFaultInjection.Delay</a>
        percent: int32
        fixedDelay: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
      abort: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPFaultInjection-Abort">HTTPFaultInjection.Abort</a>
        percent: int32
        httpStatus: int32
    mirror: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Destination">Destination</a>
      host: string
      subset: string
      port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#PortSelector">PortSelector</a>
        number: uint32
    corsPolicy: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#CorsPolicy">CorsPolicy</a>
      allowOrigin: string[]
      allowMethods: string[]
      allowHeaders: string[]
      exposeHeaders: string[]
      maxAge: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
      allowCredentials: bool
    appendHeaders: map&lt;string, string&gt;
  tls: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TLSRoute">TLSRoute[]</a>
    match: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TLSMatchAttributes">TLSMatchAttributes[]</a>
    - sniHosts: string[]
      destinationSubnets: string[]
      port: uint32
      sourceLabels: map&lt;string, string&gt;
      gateways: string[]
    route: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#DestinationWeight">DestinationWeight[]</a>
    - destination: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Destination">Destination</a>
        host: string
        subset: string
        port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#PortSelector">PortSelector</a>
          number: uint32
      weight: int32
  tcp: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TCPRoute">TCPRoute</a>
    match: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#L4MatchAttributes">L4MatchAttributes[]</a>
    - destinationSubnets: string[]
      port: uint32
      sourceLabels: map&lt;string, string&gt;
      gateways: string[]
    route: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#DestinationWeight">DestinationWeight[]</a>
    - destination: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Destination">Destination</a>
      host: string
        subset: string
        port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#PortSelector">PortSelector</a>
          number: uint32
      weight: int32  
</pre>

## DestinationRule

<pre>
apiVersion: networking.istio.io/v1alpha3
kind: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#DestinationRule">DestinationRule</a>
metadata:
  name: bookinfo-ratings
spec:
  host: string
  trafficPolicy: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TrafficPolicy">TrafficPolicy</a>
    loadBalancer: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#LoadBalancerSettings">LoadBalancerSettings</a>
      simple: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#LoadBalancerSettings-SimpleLB">LoadBalancerSettings.SimpleLB</a>
      consistentHash: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#LoadBalancerSettings-ConsistentHashLB">LoadBalancerSettings.ConsistentHashLB</a>
        httpHeaderName: string (oneof)
        httpCookie: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#LoadBalancerSettings-ConsistentHashLB-HTTPCookie"> LoadBalancerSettings.ConsistentHashLB.HTTPCookie(oneof)</a>
        useSourceIp: bool (oneof)
        minimumRingSize: uint64
    connectionPool: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ConnectionPoolSettings">ConnectionPoolSettings</a>
      tcp: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ConnectionPoolSettings-TCPSettings">ConnectionPoolSettings.TCPSettings</a>
        maxConnections: int32
        connectTimeout: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
      http: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ConnectionPoolSettings-HTTPSettings">ConnectionPoolSettings.HTTPSettings</a>
        http1MaxPendingRequests: int32
        http2MaxRequests: int32
        maxRequestsPerConnection: int32
        maxRetries: int32
    outlierDetection: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#OutlierDetection">OutlierDetection</a>
      consecutiveErrors: int32
      interval: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
      baseEjectionTime: <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">google.protobuf.Duration</a>
      maxEjectionPercent: int32
    tls: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TLSSettings">TLSSettings</a>
      mode: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TLSSettings-TLSmode">TLSSettings.TLSmode</a>
      clientCertificate: string
      privateKey: string
      caCertificates: string
      subjectAltNames: string[]
      sni: string
    portLevelSettings: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TrafficPolicy-PortTrafficPolicy">TrafficPolicy.PortTrafficPolicy[]</a>
      port: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#PortSelector">PortSelector</a>
        number: uint32
      loadBalancer: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#LoadBalancerSettings">LoadBalancerSettings</a>
      connectionPool: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ConnectionPoolSettings">ConnectionPoolSettings</a>
      outlierDetection: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#OutlierDetection">OutlierDetection</a>
      tls: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TLSSettings">TLSSettings</a>
  subsets: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Subset">Subset[]</a>
  - name: string
    labels: map&lt;string, string>&gt;
    trafficPolicy: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#TrafficPolicy">TrafficPolicy</a>
</pre>

## ServiceEntry

<pre>
apiVersion: networking.istio.io/v1alpha3
kind: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry">ServiceEntry</a>
metadata:
  name: external-svc-mongocluster
spec:
  hosts: string[]
  addresses: string[]
  ports: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Port">Port[]</a>
  - name: string
    number: uint32
    protocol: string
  location: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry-Location">ServiceEntry.Location</a>
  resolution: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry-Resolution">ServiceEntry.Resolution</a>
  endpoints: <a href="https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry-Endpoint">ServiceEntry.Endpoint[]</a>
  - address: string
    ports: map&lt;string, uint32&gt;
    labels: map&lt;string, string&gt;
</pre>

[1]: https://istio.io/docs/reference/config/istio.networking.v1alpha3/
