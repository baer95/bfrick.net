# My personal website

Available at: https://bfrick.net/

Built with https://github.com/gohugoio/hugo and https://github.com/luizdepra/hugo-coder/.

## Development

Run this for a local development server with live reload:

```shell
hugo server
```

## Deployment

See [.github/workflows/pages.yaml](.github/workflows/pages.yaml)

## Updating themes

The themes in the `themes` directory are git submodules. To update them, run the
following commands:

```shell
# initialize the submodules 
git submodule update --init --recursive

# update submodules to their latest versions
git submodule update --recursive --remote

# push the changes
git commit -m "update themes"
git push
```
