# Blog
A personal blog build with Hugo and integrate with Fleek.
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
This repo integrate with github action, so just run `hugo` and then push your repo. It will automatically deploy `/pulbic` to IPFS and update the IPNS.