# Streamlined Ecosystem of Bitcoin Apps

The following sketches an ideal of a streamlined ecosystem of apps to accompany users savely thoughout the world of Bitcoin.
It can be seen as a thought experiment to navigate the community's development efforts. Ideally, it contributes to the development of more explicit usability standards within Bitcoin's ecosystem.

It makes sense to distinguish three main Bitcoin applications:
- Wallet
- Safe
- Vault

With the following characteristics:

|                | Wallet                  | Safe                         | Vault                                             |
|----------------|-------------------------|------------------------------|---------------------------------------------------|
| Purpose        | day-to-day spending     | savings, large expenditures  | savings                                           |
| Risk           | losing some money       | losing months of salary      | such a loss would ruin most people's lives        |
| Amounts        | ~$500                   | ~$10'000                     | ~$100'000                                         |
| Key Management | phone + paper backup    | hardware wallet + desktop PC | multi-signature, hardware wallets, air-gapped PCs |
| Attributes     | quick, cheap, simple    | secure, save, simple         | most secure, most save, as simple as possible     |
| Layers         | LN, BN, "Bitcoin Banks" | LN, BN                       | BN                                                |
| Setup Cost     | none                    | low                          | medium                                            |


Ideally built upon an ecosystem of components: 
- A consistent *Security Design Language* throughout the full user experience. ( i.e. a Material Design dialect )
- A common code base to develop high quality apps ( Electron / React native / PWA  )
  - Libraries of security UI components ( HTML, CSS, JS )
  - Libraries of Bitcoin tools for transacting and networking ( RUST + WebAssembly ? )
- Privacy tools seamlessly integrated by default ( TOR, CoinJoin, ... )
- Seamlessly integrated p2p exchange ( in person & online )
- Self-serving network of lite nodes sharing succinct proofs for the global consensus

Most of these components already exist today in some way. Still, many apps have a cumbersome usability. 
I would advocate to develop industry standards that provide user experiences comparable to polished Apple or Google products.

Simplicity of financial applications is not just a mere design issue like picking a nice color for your buttons. 
Simplicity is fundamental to security because it reduces room for error. 
