# Nix binary cache container

An NGINX caching proxy to serve the cache.nixos.org.

## Usecase

I have a fast smaller SSD, but a large slower spinning HDD. I wanted to have a "cache" on the slower HDD - which allows me to garbage collect my nix store much more frequently.

That way if I do actually need a package again, it retrieves my "slow HDD" cache and not the external "http://cache.nixos.org/".

The nginx caching proxy transparently fetches any package from the upstream cache.nixos.org on first access, and saves it to its local disk so that subsequent accesses don't hit the internet.

## Instructions

1. Add the container from `nginx-binary-cache-proxy.nix` to your Nixos configuration. Also enable "NAT" on your Nixos Host.

2. Bind mount the nginx's cache path onto a path on your slower HDD: 

Within hardware-configuration.nix

```
  fileSystems."/var/lib/containers/nixbincache/var/public-nginx-cache" =
    { device = "/home/chris/mount/raid18t/Ordered/NixpkgsCache";
      fsType = "none";
      options = [ "bind" ];
    };
```

3. Simply modify the host's binary caches sources with:

```nix
{
  nix.binaryCaches = [
    "http://192.168.140.10"
    "http://cache.nixos.org/" # include this line if you want it to fallback to upstream if your cache is down
  ];
}
```

For non-NixOS nix users, set the `binary-caches` option in `/etc/nix/nix.conf` as described in the last paragraphs of [this manual section](https://nixos.org/nix/manual/#ssec-binary-cache-substituter).

Note we're using plain `http` here, which is safe because nix packages are signed with public-key cryptography.
If you care to have a bit more privacy (a man-in-the-middle not trivially observing what packages are downloaded; but most people don't care if somebody knows what publicly available packages they install) and can tolerate more roundtrips for connection initialisation (which nix < 1.12 does for each package), use `https` here instead.
