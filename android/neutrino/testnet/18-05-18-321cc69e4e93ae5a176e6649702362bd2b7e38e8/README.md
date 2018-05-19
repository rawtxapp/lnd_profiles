# Information from pprof about bottlenecks:

## 2-background syncing:
While syncing, for cpu, almost half of time is spent in double hashing.
```
(pprof) list DoubleHashH
Total: 4.28s
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/chaincfg/chainhash.DoubleHashH in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/chaincfg/chainhash/hashfuncs.go
         0         2s (flat, cum) 46.73% of Total
         .          .     26:}
         .          .     27:
         .          .     28:// DoubleHashH calculates hash(hash(b)) and returns the resulting bytes as a
         .          .     29:// Hash.
         .          .     30:func DoubleHashH(b []byte) Hash {
         .      1.31s     31:	first := sha256.Sum256(b)
         .      690ms     32:	return Hash(sha256.Sum256(first[:]))
         .          .     33:}
```

For memory, most allocations seem to happen here:
```
(pprof) list BtcDecode
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/wire.(*MsgHeaders).BtcDecode in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/wire/msgheaders.go
    1.20MB     1.20MB (flat, cum) 22.11% of Total
         .          .     49:		return messageError("MsgHeaders.BtcDecode", str)
         .          .     50:	}
         .          .     51:
         .          .     52:	// Create a contiguous slice of headers to deserialize into in order to
         .          .     53:	// reduce the number of allocations.
    1.20MB     1.20MB     54:	headers := make([]BlockHeader, count)
         .          .     55:	msg.Headers = make([]*BlockHeader, 0, count)
         .          .     56:	for i := uint64(0); i < count; i++ {
         .          .     57:		bh := &headers[i]
         .          .     58:		err := readBlockHeader(r, pver, bh)
         .          .     59:		if err != nil {
(pprof) list loadS256BytePoints
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/btcec.loadS256BytePoints in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/btcec/precompute.go
    1.11MB     1.11MB (flat, cum) 20.42% of Total
         .          .     39:		return err
         .          .     40:	}
         .          .     41:
         .          .     42:	// Deserialize the precomputed byte points and set the curve to them.
         .          .     43:	offset := 0
    1.11MB     1.11MB     44:	var bytePoints [32][256][3]fieldVal
         .          .     45:	for byteNum := 0; byteNum < 32; byteNum++ {
         .          .     46:		// All points in this window.
         .          .     47:		for i := 0; i < 256; i++ {
         .          .     48:			px := &bytePoints[byteNum][i][0]
         .          .     49:			py := &bytePoints[byteNum][i][1]
(pprof) list deserializePeers
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/addrmgr.(*AddrManager).deserializePeers in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcd/addrmgr/addrmanager.go
  512.22kB   512.22kB (flat, cum)  9.22% of Total
         .          .    484:
         .          .    485:			if ka.refs == 0 {
         .          .    486:				a.nNew++
         .          .    487:			}
         .          .    488:			ka.refs++
  512.22kB   512.22kB    489:			a.addrNew[i][val] = ka
         .          .    490:		}
         .          .    491:	}
         .          .    492:	for i := range sam.TriedBuckets {
         .          .    493:		for _, val := range sam.TriedBuckets[i] {
         .          .    494:			ka, ok := a.addrIndex[val]
(pprof) list syncWithChain
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcwallet/wallet.(*Wallet).syncWithChain in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/roasbeef/btcwallet/wallet/wallet.go
  512.03kB   512.03kB (flat, cum)  9.22% of Total
         .          .    481:			}
         .          .    482:
         .          .    483:			err = w.Manager.SetSyncedTo(ns, &waddrmgr.BlockStamp{
         .          .    484:				Hash:      *hash,
         .          .    485:				Height:    height,
  512.03kB   512.03kB    486:				Timestamp: timestamp,
         .          .    487:			})
         .          .    488:			if err != nil {
         .          .    489:				tx.Rollback()
         .          .    490:				return err
         .          .    491:			}
(pprof) list Rotator.Run
Total: 5.42MB
(pprof) list logrotate
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/jrick/logrotate/rotator.(*Rotator).Run in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/jrick/logrotate/rotator/rotator.go
         0      514kB (flat, cum)  9.26% of Total
         .          .     84:// should not be called concurrently with Write.
         .          .     85://
         .          .     86:// Prefer to use Rotator as a writer instead to avoid unnecessary scanning of
         .          .     87:// input, as this job is better handled using io.Pipe.
         .          .     88:func (r *Rotator) Run(reader io.Reader) error {
         .      514kB     89:	in := bufio.NewReader(reader)
         .          .     90:
         .          .     91:	// Rotate file immediately if it is already over the size limit.
         .          .     92:	if r.size >= r.threshold {
         .          .     93:		if err := r.rotate(); err != nil {
         .          .     94:			return err
(pprof) list handleHeadersMsg
Total: 5.42MB
ROUTINE ======================== github.com/lightningnetwork/lnd/vendor/github.com/lightninglabs/neutrino.(*blockManager).handleHeadersMsg in /ext-go/1/src/github.com/lightningnetwork/lnd/vendor/github.com/lightninglabs/neutrino/blockmanager.go
  596.16kB   596.16kB (flat, cum) 10.74% of Total
         .          .    913:			// Finally initialize the header ->
         .          .    914:			// map[filterHash]*peer map for filter header
         .          .    915:			// validation purposes later.
         .          .    916:			e := b.headerList.PushBack(&node)
         .          .    917:			b.mapMutex.Lock()
  596.16kB   596.16kB    918:			b.basicHeaders[node.header.BlockHash()] = make(
         .          .    919:				map[chainhash.Hash][]*ServerPeer,
         .          .    920:			)
         .          .    921:			b.extendedHeaders[node.header.BlockHash()] = make(
         .          .    922:				map[chainhash.Hash][]*ServerPeer,
         .          .    923:			)
```

Even though these are the bottlenecks, the numbers seem reasonable for what
they are doing.

## 3-foreground synced:
Since the UI layer is currently polling LND, it seems like it's spending quite
a bit of time/cpu in server handlers, etc.
