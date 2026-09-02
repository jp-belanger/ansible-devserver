# Ansible development server

The Ansible repository stays on the local workstation. Only the dotfiles and project
repositories are cloned onto the server.

## Prerequisites

Install the full Ansible distribution on the local workstation and clone this
repository there. Connect to the new server once as root to verify SSH access
and accept its host key:

```shell
ssh root@SERVER
# Keep this session open until the admin account and sudo have been verified.
```

Replace `SERVER` in the commands below with the server's hostname or address.
The trailing comma in the inline inventory is required.

## Bootstrap the administrator remotely

Run from the repository checkout on the local workstation:

```shell
ansible-playbook -i 'SERVER,' -u root bootstrap.yml
```

Arch Minimal may not include Python. The bootstrap first performs a full system
upgrade and installs Python and sudo using Ansible's `raw` module. It then
prompts twice for the new `admin` password, creates the account, authorizes the
configured SSH key, and grants password-protected sudo access.

Keep the root session available until the administrator login and sudo access
have been verified:

```shell
ssh admin@SERVER
sudo -v
exit
```

## Provision the development server remotely

Run from the local workstation:

```shell
ansible-playbook -i 'SERVER,' -u admin server.yml \
  --tags install \
  --ask-become-pass
```

The main playbook creates the unprivileged `dev` account, removes its
supplementary groups (including `wheel`), and authorizes the configured SSH
key. System changes run through sudo; Git, dotfile, Neovim, and Rust user
configuration runs as `dev`.

## Install yay

Yay must be built as a non-root user. Connect as `admin` after provisioning:

```shell
ssh admin@SERVER
sudo pacman -Syu --needed git base-devel

git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd .. && rm -rf yay
```

## Connect Tailscale

Tailscale is machine-wide and only needs to be connected once.
```json
{
  "tagOwners": {
    "tag:server": ["autogroup:admin"]
  },
  "grants": [
    {
      "src": ["autogroup:member"],
      "dst": ["tag:server"],
      "ip": ["tcp:22"]
    }
  ],
  "ssh": [
    {
      "action": "check",
      "src": ["autogroup:member"],
      "dst": ["tag:server"],
      "users": ["admin", "dev"]
    }
  ]
}
```

Here, Tailscale autogroups refer to tailnet identities, not Linux accounts. On
a multi-user tailnet, replace `autogroup:member` with a restricted Tailscale
group. The explicit `users` list prevents access to arbitrary local accounts.

Then connect the machine, request the tag, and enable Tailscale SSH:

```shell
sudo tailscale up --ssh --advertise-tags=tag:server
```

Confirm that `tag:server` appears on the machine in the Tailscale admin console.
Assigning a tag gives the machine a tag-based identity instead of a user-based
identity, which is appropriate for a server.

The normal development login over Tailscale is then:

```shell
tailscale ssh dev@SERVER
```

Tailscale SSH only handles connections arriving over Tailscale. The regular
OpenSSH daemon continues to handle SSH on non-Tailscale interfaces.

## Setup checklist

- [ ] Install Ansible and clone this repository on the local workstation.
- [ ] Connect as `root` and keep that session open.
- [ ] Run `bootstrap.yml` remotely as `root`.
- [ ] Connect in a second terminal as `admin` and verify `sudo -v`.
- [ ] Close the initial root session only after admin access works.
- [ ] Run `server.yml` remotely as `admin`.
- [ ] Install yay while connected as `admin`.
- [ ] Define the `tag:server`, network grant, and SSH rule in the tailnet policy.
- [ ] Run `sudo tailscale up --ssh --advertise-tags=tag:server` as `admin`.
- [ ] Verify `tag:server` in the Tailscale admin console.
- [ ] Verify `tailscale ssh dev@SERVER` from the local workstation.
