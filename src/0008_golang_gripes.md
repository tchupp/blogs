# Things that make me uneasy about Golang
1. The build tool chain is immature. The language has been around since 2009 and there is still no standard way to build or run tests in a single command (no npm run build or cargo build parallel)

1. We spend more time reading code than writing it. Go uses this to justify the lack of generics, but to get around this most libraries/SDKs will up-cast things that _could_ be generic to interface{}. This kinda defeats some of the major benefits of using a strongly-typed compiled language...

1. To add to "read more than write" point gripe, go uses CaPiTaLiZaTiOn instead of keywords to indicate public vs private scope. This makes it look like your code is written by someone who has no idea what a style guide is, and just makes Types/Properies/Functions capital or not at random.
