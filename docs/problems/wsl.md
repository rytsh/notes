# WSL

## Cannot execute windows binaries

https://github.com/microsoft/WSL/issues/8843#issuecomment-1459120198

```sh
sudo sh -c 'echo :WSLInterop:M::MZ::/init:PF > /usr/lib/binfmt.d/WSLInterop.conf'
```

After shutdown and restart WSL, you can execute windows binaries.

```sh
wsl --shutdown
```
