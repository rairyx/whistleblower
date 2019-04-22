# Raven

## Anonymous decentralized messaging network based on libp2p and dandelion

Anonymity is achieved by implementing Dandelion protocol on top of libp2p's pub/sub module, dandelion is a privacy preserving protocol to make message sender anonymous, it has 2 phases, the first phase is stem phase, where messages go through a psuedo-random path, the second phase is fluffing, at a random time of the stem phase, the message is diffused to its surrounding peers, so the third party observer cannot track back the node original node who send the message, because the message is relayed through an anonymous graph.
Message broadcasting is implemented by libp2p floodsub. 

**Dandelion implementation on libp2p-pubsub**: https://github.com/rairyx/go-libp2p-pubsub/tree/dandelion


## Demo

**Directory**:  `pubsub`

**What it demonstrates**:  Three Go peers, one JS peer are all created and run a chat server using a shared PubSub topic.  Typing text in any peer sends it to all the other peers.

**Quick test**:  `cd pubsub` and then run `./test/test.sh`.  Requires Terminator (eg, `sudo apt-get install terminator`).  The rest of this section describes how to test manually.


**First terminal**:  Create the bootstrapper node

```
cd pubsub
./pubsub-interop ../util/private_key.bin.bootstrapper.Wa --bootstrapper
```

The bootstrapper creates a new libp2p node, subscribes to the shared topic string, spawns a go routine to emit any publishes to that topic, and then waits forever.

(Note that the node ID of `pubsub-interop` is going to be `Qm...6aJ9oRuEzWa`.  Node IDs in libp2p are just public keys, and the public key `Qm...6aJ9oRuEzWa` is derived from the private key file `../util/private_key.bin.bootstrapper.Wa`.  That file is just an X.509 keypair generated by the included program `util/private-key-gen`).  We use fixed public/private keypairs for each node in this example to keep things simple.)

**Second terminal**:  Create a go peer to connect to bootstrapper and publish on the topic

```
cd pubsub
./pubsub-interop ../util/private_key.bin.peer.Sk
```

This peer, which is not in bootstrapper mode, creates a node, subscribes to the shared topic string, spawns the same go routine, and then loops forever requesting user input and publishing each line to the topic.

**Third terminal**:  Create another go peer to connect to bootstrapper and publish on the topic

```
cd pubsub
./pubsub-interop ../util/private_key.bin.peer.vy
```



**Fourth terminal**:  Create a JS peer to connect to bootstrap and publish on topic

```
cd pubsub/js
npm install  # first time only
node index.js /ip4/127.0.0.1/tcp/5555/ipfs/QmehVYruznbyDZuHBV4vEHESpDevMoAovET6aJ9oRuEzWa
```

This JS peer will accept lines of text typed on stdin, and publish them on the PubSub topic.

(Note that the JS peer generates a new identity (public/private keypair) each time, and prints its public key to stdout.  This is a deficiency in the demo; to be consistent with the Go code it should accept a private key on the CLI.)


If you return to the second, third or fourth terminals and type a message, the bootstrapper and the other 2 peers will all print your message.

**Conclusion**

You now have a chat app on a private libp2p network where each node can exchange messages anonymously using PubSub.

## Debugging Notes

**JS** To see debug messages from the Node.js program, use the `DEBUG` environment variable:
```
DEBUG="libp2p:floodsub*,libp2p:switch*,mss:*" node index.js [args...]
```

**Go** To see debug messages in Go programs, do this at runtime:
```
IPFS_LOGGING=debug ./pubsub-interop [args...]
```

(**TODO**:  describe custom instrumenting the local go code for complex debugging)

If you instrument your go code with custom `fmt.Println`'s, then revert back like this:
```
cd $GOPATH
go get -u ./...
```

Other useful commands:
```
go get -u github.com/libp2p/go-libp2p-kad-dht   # fetch just Kad DHT repo
```


_Acknowledgements:  @jhiesey for DHT (content & peer routing) JS+Go interop, @stebalien for PubSub_

