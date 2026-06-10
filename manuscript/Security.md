# Optional Material: Security Analysis

I evaluated the security risks of `little-coder`'s built in tools and I feel comfortable using the setup described in this book on my development laptop that doesn't contain my financial or other sensitive data.

That said I asked Gemini 3.5 Pro to perform a security analysis that I present here:

	1.	Framework Identification: little-coder is an autonomous coding agent framework (built on top of the pi architecture) optimized for local inference. Its primary design paradigm is to operate directly on a local codebase.
	2.	Capability Mapping: The core primitives exposed to the model by little-coder include file manipulation (Read/Write/Edit) and Bash execution. It is explicitly intended to write code, compile it, run tests, and manage project states via shell commands.
	3.	Threat Vector Continuity: Swapping the Hermes framework for little-coder while maintaining a bare-metal macOS execution environment does not alter the underlying security architecture. The threat model is identical: a probabilistic engine is driving a process equipped with the user's native system privileges.
	4.	Model Profile: gemma4:12b-it-qat possesses advanced tool-calling and code-generation capabilities. In the event of a hallucination or an adversarial payload, it has the syntactic competence to generate and execute destructive or compromised shell scripts.

## Bare-Metal Security Risks for little-coder

Running little-coder natively is functionally equivalent to granting an unverified, automated script engine user-equivalent access to your user space. The possible vulnerabilities are severe and intrinsic to its core design:

•	Arbitrary Bash Execution: Because little-coder is designed to run shell commands to verify its outputs, any hallucinated command executes directly against your macOS kernel. An incorrect path calculation during a build step or a malformed deletion command will instantly impact your host OS.
•	Supply Chain & Prompt Injection: Coding agents frequently pull external dependencies, read README files, and scrape documentation. If an external package, cloned repository, or ingested file contains an adversarial prompt injection, the agent can be hijacked to execute malicious code natively.
•	Unrestricted File System Traversal: While little-coder defaults to operating within its working directory, a path traversal error or hallucination (e.g., accessing ../../../.ssh/id_rsa) allows the model to read, modify, or exfiltrate any file accessible by your macOS user account.
•	Rogue Process Spawning: If the agent is tasked with testing a network service or server implementation, it can bind to local host ports or spawn detached background daemon processes that persist silently after the agent session is terminated.

# Optional Material: Running on macOS using a Non Priveleged Account

While not as secure as using Docker or Apple Containers, I often set up a new account on my Mac without admin priveleges and no access to my iCloud account (so sensitive financial or medical data in, for example, ~/Documents is not accessible).

I consider this to be an approach for partial security and but I use it when experimenting with `Hermes Agent`, `little-coder`, etc.

If you want tighter security, then use the material in the following chapter on Apple Containers.

Dear reader, decide for yourself how secure you want your use of agentic coding to be.
  

# Optional Material: Using Apple Containers

Here we look at a setup that is more secure than simply running in a non privileged macSO account as we did in the previous chapter.

## Create the Dockerfile

Create a file named Dockerfile to provision an Ubuntu image with systemd and Ollama. The specific systemctl mask commands follow Apple's documented recommendations for their container environment to ensure the VM boots quickly without hanging on unavailable hardware interfaces.

```
FROM ubuntu:24.04
ENV container container

# 1. Install base OS management dependencies
RUN apt-get update && \
    apt-get install -y \
    dbus systemd openssh-server net-tools iproute2 iputils-ping \
    curl wget vim-tiny man sudo pciutils zstd build-essential gnupg

# 2. Inject NodeSource PPA and install Node.js 22 + npm
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
    apt-get install -y nodejs

# 3. Download and install Ollama Linux binary
RUN curl -fsSL https://ollama.com/install.sh | sh

# 4. Install the little-coder framework globally
RUN npm install -g little-coder

# Clean up apt caches to minimize image size
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# 5. Systemd configuration for Apple Host Integration
RUN >/etc/machine-id && >/var/lib/dbus/machine-id
RUN systemctl set-default multi-user.target
RUN systemctl mask \
    dev-hugepages.mount \
    sys-fs-fuse-connections.mount \
    systemd-update-utmp.service \
    systemd-tmpfiles-setup.service \
    console-getty.service
RUN systemctl disable networkd-dispatcher.service
RUN sed -i -e 's/^AcceptEnv LANG LC_\*$/#AcceptEnv LANG LC_*/' /etc/ssh/sshd_config
```

## Use the container CLI to build the OCI image locally

```
container build -t local/ollama-machine:latest .
```

```
container machine create local/ollama-machine:latest --name little-coder-env
container machine set -n little-coder-env cpus=8 memory=16G
```

```
container machine run -n little-coder-env -- little-coder --model ollama/gemma4:12b-it-qat
```

```
$ container machine stop little-coder-env
$ container machine run -n little-coder-env

### Deleting the container
container machine rm little-coder-env

# 1. Gracefully halt the underlying Linux kernel
container machine stop little-coder-env

# 2. Drop into an interactive shell to trigger a fresh boot cycle
container machine run -n little-coder-env

# 3. Enable and start the systemd service layer for Ollama inside the VM
sudo systemctl enable ollama
sudo systemctl start ollama

# 4. Check to verify the daemon is active and listening
sudo systemctl status ollama

# 5. Pull the newly released Gemma 4 compression-optimized model directly
ollama pull gemma4:12b-it-qat

# Tada! Run little-coder:
little-coder --model ollama/gemma4:12b-it-qat

# after working in your little-coder session, use exit to leave the container and stop the container:

exit
container machine stop little-coder-env
```

You only need to download the model once to the container.
