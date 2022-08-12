# Personal Blog & Portfolio ✈️


This repository contains the code for my personal website.

[Click Here To View My Blog \[https://AustinChristiansen.com\]](https://austinchristiansen.com)

## Local Development
1. Install Hugo with `brew install hugo` or your preferred package manager
2. `git submodule update --init --recursive` - Pulls in forked theme as submodule for Hugo
3. `npm i` - Install dependencies
4. `npm run dev` - Starts development server on a random port

## Production
`npm run build`

## Deployments
Deployments are automated and deployed to Github pages using Github actions when commits are merged into master.

## Security & Caching
This site is protected using [Cloudflare](https://cloudflare.com) SSL. Additionally, this blog is leveraging [Cloudflare](https://cloudflare.com) cache for page load optimization and their generous edge caching. 