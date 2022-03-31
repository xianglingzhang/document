# Naor-Pinkas Oblivious Transform
`naor_pinkas_ot` is an implementation of [Efficient Oblivious Transfer Protocols](https://dl.acm.org/doi/10.5555/365411.365502). It also plays the role of base ot in [IKNP OT extension](iknp_ote.md). 

## Construction
There are two parties of the tow-choose-one OT protocol, sender and receiver. Sender has input {m_0, m_1}, receiver has an input choice bit b; the output of the sender is nothing, while the output of receiver is m_b.

### Public Parameters
```
struct PP
{
    ECPoint g;
};
```
The `PP` struct holds the public parameter of NPOT protocol, which is a generator group `g`. It can be initialized by `Setup()`. 
```
PP Setup();
```

### Serialization
```
void SerializePP(PP &pp, std::ofstream &fout);
void SavePP(PP &pp, std::string pp_filename);
```
The public parameter `PP` can be serialized and saved to file `pp_filename`. `SavePP` will call `SerializePP` internally.
```
void DeserializePP(PP &pp, std::ifstream &fin);
void FetchPP(PP &pp, std::string pp_filename);
```
Similarly, `FetchPP` will call `DeserializePP` to fetch serialized `PP` from file `pp_filename`.

### Instantiate
If we want to instantiate a number of OTs, the following function will be called by sender
```
void Send(NetIO &io, PP &pp, const std::vector<block>& vec_m0, const std::vector<block> &vec_m1, size_t LEN);
```
* `NetIO &io`: a class object handle socket communication. Sender is deemed as "server".
* `PP &pp`: public parameter of `NPOT`.
* `std::vector<block>& vec_m0`: a vector of messages. 
* `std::vector<block> &vec_m1`: another vector of messages.
* `size_t LEN`: the number of OT instances, that is, the length of vector `vec_m0` and `vec_m1`.

receiver will call
```
std::vector<block> Receive(NetIO &io, PP &pp, const std::vector<uint8_t> &vec_selection_bit, size_t LEN);
```
* `NetIO &io`: a class object handle socket communication. Receiver is deemed as "client".
* `std::vector<uint8_t> &vec_selection_bit`: a vector of choice bits. receiver will choose from `vec_m0` or `vec_m1` according to the choice bit.
* `size_t LEN`: the number of OT instances, that is, the length of vector `vec_selection_bit`; it will be the same as `NPOT::Send` input `size_t LEN`.

which will return a `std::vector<block>` result corresponding to the choice bit vector `vec_selection_bit`.

## Usage
More detailed sample code are provided in test files. Here we just show an overview.
```
PRG::Seed seed = PRG::SetSeed(nullptr, 0); // initialize PRG seed
NPOT::PP pp = NPOT::Setup(); 

std::vector<uint8_t> vec_selection_bit = GenRandomBits(seed, NUM); 
std::vector<block> vec_K0 = PRG::GenRandomBlocks(seed, NUM);
std::vector<block> vec_K1 = PRG::GenRandomBlocks(seed, NUM);

if (party == "sender")
{
    NetIO sender_io("server", "", 8080); 
    NPOT::Send(sender_io, pp, vec_K0, vec_K1, NUM); 
}

if (party == "receiver")
{
    NetIO receiver_io("client", "127.0.0.1", 8080);
    std::vector<block> vec_K = NPOT::Receive(receiver_io, pp, vec_selection_bit, NUM); 
}
```
