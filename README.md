IPFS源码分析第一章 go-libp2p-core 模块
	1. peer节点

		1. Struct  AddrInfo 数据结构

                       type AddrInfo struct {               ID ID                  //名称 （ String)               Addrs []ma.Multiaddr   //自描述网络地址           }       2. type ID string，Peer节点           操作函数:（Base58,Pretty , ....)       3. Set 集合            Peer的Set操作
	1. peerstore

		1. AddrBook
		2. KeyBook
		3. Metrics
		4. ProtoBook
		5. PeerMetadata
		6. Peerstore


	1. Crypto 加密



		1. ESDSA 椭圆曲线数字签名算法

			* 椭圆曲线数字签名算法（ECDSA）是使用椭圆曲线密码（ECC）对数字签名算法（DSA）的模拟。ECDSA于1999年成为ANSI标准，并于2000年成为IEEE和NIST标准。
			* 椭圆曲线密码体制的安全性基于椭圆曲线离散对数问题（ECDLP）的难解性。椭圆曲线离散对数问题远难于离散对数问题，椭圆曲线密码系统的单位比特强度要远高于传统的离散对数系统。因此在使用较短的密钥的情况下，ECC可以达到于DL系统相同的安全级别。这带来的好处就是计算参数更小，密钥更短，运算速度更快，签名也更加短小。因此椭圆曲线密码尤其适用于处理能力、存储空间、带宽及功耗受限的场合。
			* 详细参考：https://blog.csdn.net/m0_37458552/article/details/80250258
		2. ED25519 加密算法

			1. 详细参考： https://blog.csdn.net/u013137970/article/details/84573265
		3. RSA 加密算法

			1. 详细参考： https://blog.csdn.net/jijianshuai/article/details/80582187
		4. Sep256k1

			1. 详细参考： https://www.jianshu.com/p/7a88fd599ade
		5. Key  密钥操作(私钥，公钥）

            对上术述加密算法进行抽像操作
	1.     network      网络 

		1.  stream   stream表示中两个代理之间的双向通道,支持多路复用

            // Stream represents a bidirectional channel between two agents in// a libp2p network. "agent" is as granular as desired, potentially// being a "request -> reply" pair, or whole protocols.//// Streams are backed by a multiplexer underneath the hood.            type Stream interface {mux.MuxedStreamProtocol() protocol.IDSetProtocol(id protocol.ID)// Stat returns metadata pertaining to this stream.Stat() Stat// Conn returns the connection this stream is part of.Conn() Conn}            2. network  网络是连接外部世界的接口,常用于Swarm 网络// Network is the interface used to connect to the outside world.// It dials and listens for connections. it uses a Swarm to pool// connections (see swarm pkg, and peerstream.Swarm). Connections// are encrypted with a TLS-like protocol.type Network interface {Dialerio.Closer// SetStreamHandler sets the handler for new streams opened by the// remote side. This operation is threadsafe.SetStreamHandler(StreamHandler)// SetConnHandler sets the handler for new connections opened by the// remote side. This operation is threadsafe.SetConnHandler(ConnHandler)// NewStream returns a new stream to given peer p.// If there is no connection to p, attempts to create one.NewStream(context.Context, peer.ID) (Stream, error)// Listen tells the network to start listening on given multiaddrs.Listen(...ma.Multiaddr) error// ListenAddresses returns a list of addresses at which this network listens.ListenAddresses() []ma.Multiaddr// InterfaceListenAddresses returns a list of addresses at which this network// listens. It expands "any interface" addresses (/ip4/0.0.0.0, /ip6/::) to// use the known local interfaces.InterfaceListenAddresses() ([]ma.Multiaddr, error)// Process returns the network's ProcessProcess() goprocess.Process}Swarm 是一个去中心化的内容存储和分发服务。3.Dialer 拨号程序表示可以拨出到对等方的服务// Dialer represents a service that can dial out to peers// (this is usually just a Network, but other services may not need the whole// stack, and thus it becomes easier to mock)type Dialer interface {// Peerstore returns the internal peerstore// This is useful to tell the dialer about a new address for a peer.// Or use one of the public keys found out over the network.Peerstore() peerstore.Peerstore// LocalPeer returns the local peer associated with this networkLocalPeer() peer.ID// DialPeer establishes a connection to a given peerDialPeer(context.Context, peer.ID) (Conn, error)// ClosePeer closes the connection to a given peerClosePeer(peer.ID) error// Connectedness returns a state signaling connection capabilitiesConnectedness(peer.ID) Connectedness// Peers returns the peers connectedPeers() []peer.ID// Conns returns the connections in this NetowrkConns() []Conn// ConnsToPeer returns the connections in this Netowrk for given peer.ConnsToPeer(p peer.ID) []Conn// Notify/StopNotify register and unregister a notifiee for signalsNotify(Notifiee)StopNotify(Notifiee)}4.conn// Conn is a connection to a remote peer. It multiplexes streams.// Usually there is no need to use a Conn directly, but it may// be useful to get information about the peer on the other side:// stream.Conn().RemotePeer()type Conn interface {io.CloserConnSecurityConnMultiaddrs// NewStream constructs a new Stream over this conn.NewStream() (Stream, error)// GetStreams returns all open streams over this conn.GetStreams() []Stream// Stat stores metadata pertaining to this conn.Stat() Stat}// ConnSecurity is the interface that one can mix into a connection interface to// give it the security methods.type ConnSecurity interface {// LocalPeer returns our peer IDLocalPeer() peer.ID// LocalPrivateKey returns our private keyLocalPrivateKey() ic.PrivKey// RemotePeer returns the peer ID of the remote peer.RemotePeer() peer.ID// RemotePublicKey returns the public key of the remote peer.RemotePublicKey() ic.PubKey}// ConnMultiaddrs is an interface mixin for connection types that provide multiaddr// addresses for the endpoints.type ConnMultiaddrs interface {// LocalMultiaddr returns the local Multiaddr associated// with this connectionLocalMultiaddr() ma.Multiaddr// RemoteMultiaddr returns the remote Multiaddr associated// with this connectionRemoteMultiaddr() ma.Multiaddr    }        
	1. 术语

		1. Multiaddr 自描述网络地址

			* Multiaddr旨在使网络成为未来的证明，可以组合和高效。

				1. 目前的寻址方案有许多问题。
				2. 它们阻碍协议之间的协议迁移和互操作性。
				3. 它们没有很好的构造，但是很少有X-over-Y构造，但只能在经典 uri/url或者 host:port 方案中解决。
				4. 它们不是多路复用的：它们是地址端口，而不是进程。
				5. 它们是隐式的，因为它们假设out-of-band值和上下文。
				6. 它们没有高效的机器可读表示。
			* Multiaddr通过将网络地址建模为任意的协议封装来解决这些问题。

				1. 任何网络协议的Multiaddrs支持地址。
				2. Multiaddrs是自描述的。
				3. Multiaddrs符合简单语法，这使得解析和构造它们变得简单。
				4. Multiaddrs具有人类可读和高效的机器可读表示。
				5. Multiaddrs封装良好，允许对封装层进行简单的封装和展开。
			* 规范：

				1. 支持的协议：https://github.com/multiformats/multiaddr/blob/master/protocols.cs

		1. 比特币签名

 比特币签名算法： https://www.jianshu.com/p/cd55c3d5bcc1在比特币的ECDSA算法的实现中，被签名的“消息”是交易，或更确切地说是交易中特定数据子集的哈希值（参见签名哈希类型（SIGHASH））。签名密钥是用户的私钥，结果是签名：((Sig = F{sig}(F{hash}(m), dA)))这里的：● dA 是签名私钥● m 是交易（或其部分）● Fhash 是散列函数● Fsig 是签名算法● Sig 是结果签名ECDSA数学运算的更多细节可以在ECDSA Math章节中找到。函数Fsig 产生由两个值组成的签名Sig，通常称为R和S：Sig = (R, S)签名序列化（DER）解锁脚本序列化之后：   3045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301包含以下9个元素：● 0x30表示DER序列的开始● 0x45 - 序列的长度（69字节）● 0x02 - 一个整数值● 0x21 - 整数的长度（33字节）● R-00884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb● 0x02 - 接下来是一个整数● 0x20 - 整数的长度（32字节）● S-4b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e3813● 后缀（0x01）指示使用的哈希的类型（SIGHASH_ALL）      3.  Swarm和Whisper是什么鬼？         以太坊的智能合约smart contract实现了分布式逻辑，以太坊的Swarm实现了分布式存储，以太坊的Whisper实现了分布式消息，Whisper将实现智能合约间的消息互通，届时可以实现功能更加复杂的DApp。Swarm区块链能很好地存储少量的数据。 如果你想要存储病历，销售合同或需要公开时间戳的大型文件该怎么办呢？在区块链中存储大块数据是昂贵并且不可扩展的。 Swarm 被用来解决这个问题。 Swarm 是一个去中心化的内容存储和分发服务。 您可以将它视为 CDN，但它并不是在一家公司的服务器上托管的所有 CDN，而是通过互联网在计算机上分发。 你可以像运行一个以太坊节点一样，去运行一个 Swarm 节点并连接到 Swarm 网络上。swarm是点对点文件共享，它与BitTorrent相似，但用以太币为微报酬作为激励。文件被分解成块，分配并被参与的志愿者们储存。那些为存储并为块提供服务的节点，从那些需要储存和检索数据服务的节点得到以太币作为补偿。这是不依赖于中心服务器的文件存储。https://swarm-guide.readthedocs.io/en/latest/当你将一个以太坊合约部署到区块链时，您会获得一个部署地址和一个 ABI JSON 接口（类似于 API 的合约接口）。当你希望有人使用您的合约时，你需要提供部署地址和 ABI 。 将来，ABI 会被存储在 Swarm 中，以便每个人都可以通过查看以太坊地址来查找 ABI。IPFS（星际文件系统）在概念上与 Swarm 非常相似。 它是一个去中心化的存储系统。 与以太坊没有直接关联，但可以与以太坊集成。你可以在这里查看 Swarm 和 IPFS 之间的不同: https://github.com/ethersphere/go-ethereum/wiki/IPFS-&-SWARM主要异同点：同：1，都是通用的分布式存储解决方案；2，内容分发协议；异：1，Swarm使用Ethereum的devp2p（协议多路复用，通过帧，加密，认证，握手和协议消息API标准，对等连接管理支持，节点发现进行消息交织），并充分利用其强壮性，并最显着地继承了其审计和广泛赞誉）安全属性。IPFS使用libp2p网络层，这是一种类似先进的通用p2p解决方案。2，Swarm是内容寻址块存档，而IPFS更类似于bittorrent，其内容是DHT（分布式散列表）。3，Filecoin是IPFS的姊妹项目，它为IPFS增加了激励层，并依靠自己的altchain。 在文件币区块链上检索“挖掘”的证据是一种向存储器提供持续补偿以保留内容的方案。Swarm利用智能合约的全部功能来处理注册节点并存入利息。 这允许采取惩罚性措施作为威慑。 Swarm提供了一个追踪责任的计划。4，Swarm将对区块链上的极少访问内容实施高效的自动化集体审计，并提供最后诉讼。whisperwhisper是一种信息检索协议，它允许节点间直接以一种安全的形式互发信息，并对第三方组织窥探者隐藏发送者和接收者的信息。这是不依赖于一个中心服务器的通讯管理。你可能没怎么听到过 Whisper，不过它也是在以太坊生态系统中一项有趣的技术。 它是 Dapps 之间交互的通信协议。 你可以在这里看到关于它的更多内容: https://github.com/ethereum/wiki/wiki/Whisper



