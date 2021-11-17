# ygot issue

This repository is to demo a bug (I think..) with [ygot](https://github.com/openconfig/ygot) where it silently drops leaves in a certain configuration.

Here we have a subset of IETF models, some of which have been trimmed from the original, to show routing information.

```
$ pyang -f tree ietf-routing@2018-03-13.yang ietf-ipv4-unicast-routing@2018-03-13.yang
module: ietf-routing
  +--rw routing
     +--rw router-id?                 yang:dotted-quad {router-id}?
     +--ro interfaces
     |  +--ro interface*   if:interface-ref
     +--rw control-plane-protocols
        +--rw control-plane-protocol* [type name]
           +--rw type             identityref
           +--rw name             string
           +--rw description?     string
           +--rw static-routes
              +--rw v4ur:ipv4
                 +--rw v4ur:route* [destination-prefix]
                    +--rw v4ur:destination-prefix    inet:ipv4-prefix
                    +--rw v4ur:description?          string
                    +--rw v4ur:next-hop
                       +--rw (v4ur:next-hop-options)
                          +--:(v4ur:simple-next-hop)
                          |  +--rw v4ur:outgoing-interface?   if:interface-ref
                          +--:(v4ur:special-next-hop)
                          |  +--rw v4ur:special-next-hop?     enumeration
                          +--:(v4ur:next-hop-list)
                             +--rw v4ur:next-hop-list
                                +--rw v4ur:next-hop* [index]
                                   +--rw v4ur:index                 string
                                   +--rw v4ur:outgoing-interface?   if:interface-ref
                                   +--rw v4ur:next-hop-address?     inet:ipv4-address
```

The specific leaf here is the last one - `next-hop-address`. pyang sees that but ygot does not.

```
$ go run github.com/openconfig/ygot/generator \
    -logtostderr \
    -generate_simple_unions -include_descriptions \
    -path="path/to/ygot-uses-augment/" \
    -package_name=foo \
    -output_file=foo.go \
    path/to/ygot-uses-augment/ietf-routing@2018-03-13.yang \
    path/to/ygot-uses-augment/ietf-ipv4-unicast-routing@2018-03-13.yang
```

That command generates Go code and seems successful but the generated struct for next hop is missing the `NextHopAddress` leaf.

```go
type IETFRouting_Routing_ControlPlaneProtocols_ControlPlaneProtocol_StaticRoutes_Ipv4_Route_NextHop_NextHopList_NextHop struct {
	Index	*string	`path:"index" module:"ietf-ipv4-unicast-routing"`
	OutgoingInterface	*string	`path:"outgoing-interface" module:"ietf-ipv4-unicast-routing"`
}
```