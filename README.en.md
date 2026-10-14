# Music Bar Street

[中文](README.md) · [Wireframes](https://jiekertime.github.io/music-bar-street/prototype/) · [Page descriptions](https://jiekertime.github.io/music-bar-street/ui/) · [Figma file](https://www.figma.com/community/file/1692143979739836987)

Music Bar Street is an app idea for booking musicians at local bars. A bar posts a gig, musicians apply, and the bar picks someone to talk to. A bar can also find a musician through their work and send an invitation. They agree on the date, venue, fee and performance details before confirming the booking.

I am sharing the business notes, product flow, wireframes and backend design. You can use them to build your own project. If you would like to work on it together, get in touch.

The files include [product and backend design documents](docs/README.md), [10 diagrams](design/index.html), 16 grayscale wireframes and page descriptions. The app backend has not been built. The original UI is kept locally and is not included in the current public files.

## Where to start

1. [Browse the wireframes](prototype/index.html) to follow onboarding, gig discovery, agreement and completion.
2. Read the [business notes](docs/BUSINESS_MODEL.md) and [product flow](docs/PRODUCT_SPEC.md).
3. Browse the [page descriptions](ui/index.html) or copy the [Figma file](https://www.figma.com/community/file/1692143979739836987).
4. Use the [backend notes](docs/IMPLEMENTATION_GUIDE.md) when you start building.

[Download all files](https://github.com/JiekerTime/music-bar-street/archive/refs/heads/main.zip) · [Contact me](https://github.com/JiekerTime/music-bar-street/issues/new/choose)

Most detailed notes and wireframe labels are in Chinese. The wireframes describe the proposed product; they are not a running application.

## Booking rules

Posting a gig or accepting an invitation starts a conversation. A booking is confirmed only when both sides accept the same terms. The advertised budget and agreed fee are separate. Changing the terms clears earlier confirmations. After the performance, both sides confirm completion.

For the first version, the bar and musician settle the performance fee directly. The platform keeps the booking record. Membership and a 1% service fee are early ideas to test; there is no revenue or operating marketplace behind this release. Live streaming, tips and wallets are outside the first version.

## Use the files or join the project

You can adapt the files and use them commercially. You do not need to join my team or share revenue with me. Documents, wireframes and diagrams use **CC BY 4.0**; website source uses **Apache-2.0**. Keep the required credit and licence notices. See [LICENSING.md](LICENSING.md).

If you know bars or musicians, want to try this locally, or can help with design or development, leave a note in [GitHub Issues](https://github.com/JiekerTime/music-bar-street/issues). Tell me where you are and which part you want to work on. We can discuss any partnership separately.

The proposed backend uses one Spring Boot application, PostgreSQL and Redis, with Alibaba Cloud OSS for images and VOD for video. Start with booking, then add features when they are needed.

By Ruoyi / [JiekerTime](https://github.com/JiekerTime). The [Product Hunt listing](https://www.producthunt.com/products/music-bar-street?launch=music-bar-street) is scheduled for October 15, 2026 at 12:01am Pacific Time (3:01pm China Standard Time).
