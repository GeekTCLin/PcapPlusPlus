## 笔记

### 大小端判断的便捷方法
```cpp
#if (BYTE_ORDER == LITTLE_ENDIAN)

#else

#endif
```
``BYTE_ORDER``宏定义在``endian.h``文件中

### 大小端下的IP报文首字节
```cpp
#if (BYTE_ORDER == LITTLE_ENDIAN)
	/// IP header length, has the value of 5 for IPv4
	uint8_t internetHeaderLength : 4,
	/// IP version number, has the value of 4 for IPv4
	ipVersion : 4;
#else
	/// IP version number, has the value of 4 for IPv4
	uint8_t ipVersion : 4,
	/// IP header length, has the value of 5 for IPv4
	internetHeaderLength : 4;
#endif
```

### IP分片重组
主要代码位于``IPReassembly.h``
**基类``PacketKey``用于标识一个IP分片**,该类仅声明接口，随之派生出的子类``IPv4PacketKey``与``IPv6PacketKey``
**注意：**
*ipv4分片id占``uint16_t``，而ipv6分片id占``uint32_t``*

**主要函数：**
```cpp
/// The main API that drives IPReassembly. This method should be called whenever a fragment arrives. This method
/// finds the relevant packet this fragment belongs to and runs the IP reassembly logic that is described in
/// IPReassembly.h.
/// @param[in] fragment The fragment to process (IPv4 or IPv6). Please notice that the reassembly logic doesn't
/// change or manipulate this object in any way. All of its data is copied to internal structures and
/// manipulated there
/// @param[out] status An indication of the packet reassembly status following the processing of this fragment.
/// Possible values are:
/// - The input fragment is not a IPv4 or IPv6 packet
/// - The input fragment is not a IPv4 or IPv6 fragment packet
/// - The input fragment is the first fragment of the packet
/// - The input fragment is not the first or last fragment
/// - The input fragment came out-of-order, meaning that wasn't the fragment that was currently expected (it's
///   data is copied to the out-of-order fragment list)
/// - The input fragment is malformed and will be ignored
/// - The input fragment is the last one and the packet is now fully reassembled. In this case the return value
/// will contain a pointer to the reassembled packet
/// @param[in] parseUntil Optional parameter. Parse the reassembled packet until you reach a certain protocol
/// (inclusive). Can be useful for cases when you need to parse only up to a certain layer and want to avoid the
/// performance impact and memory consumption of parsing the whole packet. Note that setting this to a protocol
/// which doesn't include the IP-Layer will result in IPReassembly not finding the IP-Layer and thus failing to
/// work properly. Default value is ::UnknownProtocol which means don't take this parameter into account
/// @param[in] parseUntilLayer Optional parameter. Parse the reassembled packet until you reach a certain layer
/// in the OSI model (inclusive). Can be useful for cases when you need to parse only up to a certain OSI layer
/// (for example transport layer) and want to avoid the performance impact and memory consumption of parsing the
/// whole packet. Note that setting this value to OsiModelPhysicalLayer will result in IPReassembly not finding
/// the IP-layer and thus failing to work properly. Default value is ::OsiModelLayerUnknown which means don't
/// take this parameter into account
/// @return
/// - If the input fragment isn't an IPv4/IPv6 packet or if it isn't an IPv4/IPv6 fragment, the return value is
/// a
///   pointer to the input fragment
/// - If the input fragment is the last one and the reassembled packet is ready - a pointer to the reassembled
///   packet is returned. Notice it's the user's responsibility to free this pointer when done using it
/// - If the reassembled packet isn't ready then nullptr is returned
Packet* processPacket(Packet* fragment, ReassemblyStatus& status, ProtocolType parseUntil = UnknownProtocol,
						OsiModelLayer parseUntilLayer = OsiModelLayerUnknown);
```