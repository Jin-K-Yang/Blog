# Blog
A personal blog build with Hugo and integrate with Fleek.
# Add post
```
hugo new posts/<post-name>.md
```
# Test
```
hugo server -b http://localhost:1313
```
# Build public file
```
hugo
```
# Fleek
## Install
```
npm install -g @fleekhq/fleek-cli
```
## Setting
```
export FLEEK_API_KEY=<Fleek API Key>
fleek site:init
```
## Deploy
```
fleek site:deploy
```
# CI/CD
This repo integrate with Fleek, so just type `hugo` and push the repo, it will automatically deploy to IPFS and update the IPNS.
