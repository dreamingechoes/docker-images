# Docker images

A set of Docker images to use in the development of my personal projects. All available at [hub.docker.com/r/dreamingechoes](https://hub.docker.com/r/dreamingechoes).

## Usage example

Let's pretend that you're developing a `Phoenix` application and you want to create a basic `Dockerfile` for development purposes. The code of you `Dockerfile` will look like this:

```
FROM dreamingechoes/elixir:1.6.1

# Install hex, rebar, and get deps
ENV MIX_ENV=dev

RUN mix local.hex --force && \
    mix local.rebar --force && \
    mix hex.info

WORKDIR /YOUR_APP_PATH
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/dreamingechoes/docker-images. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The repository is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

----------------------------

This repository was created by [dreamingechoes](https://github.com/dreamingechoes).
It adheres to its [code of conduct](https://github.com/dreamingechoes/base/blob/master/files/CODE_OF_CONDUCT.md), and uses an equivalent [license](https://github.com/dreamingechoes/base/blob/master/files/LICENSE).
