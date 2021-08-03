# Strict encoding schema

Strict encoding schema is a **library** for handling binary [strict encoding] 
rules and a **compiler** from a human-readable data structure representation, 
written on a [Parsel] language, used by [AluVM] (or its subset [Contractum] used
in RGB smart contracts) into a binary format.

Example of strict encoding schema declaration in Parsel/Contractum:
```parsel
datapkg BP

alias Hash256 :: U8^32
alias Sha256 :: Hash256
alias Sha256d :: Hash256
alias Sha256t :: Hash256
alias Ripemd160 :: U8^20

alias BlockId :: Sha256d
alias Txid :: Sha256d

alias Amount :: U64

data OutPoint :: Txid, vout U16

data ScriptPubkey :: U8^..10_000
data StackByteStr :: U8^..520
data Witness :: StackByteStr^..1_000
data SigScript :: StackByteStr^..1_000

data TxOut :: Amount, 
              ScriptPubkey
data TxIn :: OutPoint, 
             nSeq U16, 
             SigScript, 
             Witness?

data Transaction :: ver U8, 
                    inputs TxIn*,
                    outputs TxOut*,
                    lockTime U32
```

Strict encoding schema library and compiler also covers *`StenPath`* - a 
standard for addressing specific fields and entries of strict-encoded data 
inside container (you may compare StenPath to XPath for XML) and its binary
encoding which is used in `STEN` AluVM ISA extensions.

For instance, StenPath expression `$1[5].$3[1][0]` applied to `Transaction`
data structure defined above it will return first byte of second
item in witness stack of the second transaction input. In binary form StenPath
can be read like `FF 01 00  FE 05 00  FF 03 00  FE 01 00  FE 00 00`.

Example of how this encoding can be used together with `STEN` ISA in 
[AluVM assembly]:

```aluasm
.ISA  ALU, BP, STEN

.MAIN
                    ;; Load transaciton with ID to s16[1]
                    bpltx   0xbdb657fe4633d4612e4096eb2cb9f575233cb007ae3a17245d80dad76afb33b9, s16[1]
                    ;; Define lookup path
                    put     0xFF0100_FE0500_FF0300_FE0100_FF0000, s16[3]

                    ;; Parses data from s16[1] according to ses-encoding found in s16[2]
                    ;; at data path encoded at s16[3] to register u8[1]
                    parse   s16[1], s16[2], s16[3], u8[1]
```

[strict encoding]: https://strict-encoding.lnp-bp.org
[AluVM]: https://www.aluvm.org
[ParselTongue]: https://parsel-lang.org
[Contractum]: https://github.com/rgb-org/contractum-lang
[AluVM assembly]: https://github.com/pandoracore/aluasm
