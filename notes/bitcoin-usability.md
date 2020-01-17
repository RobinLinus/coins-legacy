# Bitcoin Usability 

This is a loose compilation of concepts regarding the usability of Bitcoin. 

Assuming hyperbitcoinization – a world were most people pay in bitcoin – what would the usability be like?

Certainly, user-friendly Bitcoin is hard. In general, cryptography is hard. Usability is hard. Usable cryptography is insanely hard. It is an on-going process to balance the gap between security and usability. The following is a map of Bitcoin design challenges and solutions.

## Simplicity
Simplicity is a universal design philosophy. It is a fundamental heuristic with applications ranging from cryptography to product design.
Only simple constructions provide strong security. Only simplicity works because it leaves very little room for error and manipulation.

Obviously, Bitcoin services have special security requirements and therefore, special simplicity requirements. 

Regarding mainstream adoption, the psychological need for simplicity goes even deeper, since there is a high mental cost to complexity because of our instinctive loss aversion behavior when dealing with money.
[Daniel Kahneman's "Thinking, Fast and Slow"](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow) summarizes research conducted over decades. 
It can be interpreted as a theoretical foundation to describe our cognitive need for simplicity.

Furthermore, in [Insanely Simple](https://www.amazon.com/Insanely-Simple-Obsession-Drives-Success/dp/1591846218), Ken Segall describes Apple's obsession for simplicity as the number one reason for the iPhone's success (...and later, Apple becoming world's most valuable company).
Another example: the simplicity of Google's interface was a major competitive edge during the search engine war.

Simplicity is fundamental to quality design in any area. It is important to acknowledge that simplicity as the integral driver for successful security design. 
It is not a quick fix though. It is an ongoing process of hard work because complexity naturally tends to blow up. That's why, in general, there are so few products with great usability. Still, in the long term, simplicity gives clear guidance on how to optimize Bitcoin products:

Make it more simple.

## Spectrum of Use Cases

But what does "simple" actually mean? What should the Bitcoin experience be like? A copy of PayPal or ApplePay? Somewhat, yes. The big companies have spent hundreds of millions to optimize payment experiences. They've probably learned something worth imitating.
Still, there is a wide spectrum of Bitcoin use cases and thus, a spectrum of application requirements. 

### Design Dimensions
The fundamental dimensions of Bitcoin UX design are:

- online vs physical payments
  - where are you paying? On the internet or in the real world?
- amount size: large vs small 
  - how much money are you dealing with? Loose change or life savings?
- mobile vs desktop
  - what hardware do you use?
  - how is your connection?
  - how reliable is your data storage?
- payment relations
  - who are you and who is your counter-party?
    - b2b, business to business 
    - b2c, business to customer
    - p2p, person to person
- political stability and freedom
  - Is your value transfer prone to active censorship?


Such design dimensions can open endless sub-categories of use cases. We want to keep it simple. 
Can we make an educated guess and summarize the requirements of many users? 

Indeed, we can: The [next billion users](https://www.youtube.com/watch?v=6fBS8LOpIQc) will be [on billions of (low-end) mobiles](https://www.youtube.com/watch?v=ak6Uj02DTjk) with unreliable connections. Most people will make only small sized payments simply because of the factual distribution of wealth. Relatively small payments will happen much more frequently than large payments.

What can we summarize? We have a clear main target group to focus a design on. A solution for:
- Billions of endusers
- With low-end phones
- Making small payments

Obviously this isn't enough because a bitcoin economy requires businesses. 



## Wallet vs Vault
Both businesses and individuals have two general use cases: small amounts and large amounts. Further, I'd suggest to establish the usage of clear terms like "wallet" and "vault" which are self-explanatory metaphors differentiating the apps' purposes well to new users.

**"Wallet"**:
- small amounts, lower risk
- daily spendings 
- pay as quickly as possible 
  - "there are people waiting in line behind me"
  - frictionless
  - authorize payments via TouchId, FaceID, ... 
  - similar to ApplePay
  - similar to a physical wallet 
- Overall, relaxed security requirements
- Minimum system requirements:
  - low-end phone 
  - camera to scan QR
  - unreliable connection
  - NFC is advantageous
- Transactions must be as cheap as possible. Too high tx fees excludes most commerce.

**"Vault"**:
- large amounts, high risk
- potentially, life savings 
- store value and authorize payments as securely as possible
  - Multi-Signature
  - Cold storage
  - Similar to a bank savings account 
- Privacy
- Minimum system requirements:
  - Full Node
  - Reliable connection
  - Hardware Wallets
- Overall, strict security requirements. Better be safe and let the user confirm everything twice.

To fuel mass adoption, it'd be great to have two high quality solutions. One application tailored to each of those two use cases: Small amounts and large amounts. 
Those two apps should be clearly visible. The apps should be simple. Never too simple. Still, insanely simple apps. That's what makes great quality.


## Ecosystem of Apps
Ideally, there exists a clean ecosystem of solutions which are designed to inter-operate seamlessly and cover the full spectrum of use cases. Ideally, the Bitcoin community offers an ecosystem experience like Apple or Google.

Of course, there is already a great ecosystem. It is growing and I love that. Still, I would advocate for better standards and consistency. All successful ecosystems have a clear design language. Ideally such a language accompanies the user consistently throughout the full experience. The consistent repetition of designs and usability metaphors simplifies the user experience and thus, makes it secure. 

Google invented the [Material Design Language](https://material.io/design/) for their ecosystem. Material is meant to be highly flexible and customizable so I'd suggest to define a Material Design Dialect to serve as Bitcoin's design language.
Ideally, A security design language covers also best practices and common pitfalls in Bitcoin designs. 
Such a language should be expressed both in a design specification and in high-quality libraries of UI components. In tradition of best practices like security audits, we should develop industry standards for usability audits. 

Is it possible to develop tools for an open source community with usability standards as high as Apple's or Google's? 


