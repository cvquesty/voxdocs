# puppet ssl

> *Managing the PKI that underpins all OpenVox communication.*

[← Back to CLI Reference](README.md)

---

Manage SSL certificates and keys for agent-server authentication.

```console
$ puppet help ssl

puppet-ssl(8) -- Manage SSL keys and certificates for OpenVox SSL clients
========

SYNOPSIS
--------
Manage SSL keys and certificates for clients needing
to communicate with an OpenVox infrastructure.

USAGE
-----
puppet ssl <action> [-h|--help] [-v|--verbose] [-d|--debug] [--localca] [--target CERTNAME]

OPTIONS
-------
* --help:            Print this help message.
* --verbose:         Print extra information.
* --debug:           Enable full debugging.
* --localca:         Also clean the local CA certificate and CRL.
* --target CERTNAME: Clean the specified device certificate instead of this host's certificate.

ACTIONS
-------
* bootstrap:        Perform all steps to request and download a client certificate.
                    If autosigning is disabled, puppet will wait every `waitforcert`
                    seconds for its certificate to be signed. Specify 0 to never wait.
* submit_request:   Generate a CSR and submit it to the CA.
* generate_request: Generate a CSR (but don't submit it).
* download_cert:    Download a signed certificate for this host.
* verify:           Verify the private key and certificate match, and the cert is trusted.
* clean:            Remove the private key and certificate files for this host.
                    With --localca, also remove the local CA cert and CRL bundle.
                    With --target, clean a specific device certificate.
* show:             Print the full-text version of this host's certificate.

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

## Common Usage Patterns

```bash
# Bootstrap SSL (generate key + CSR, request signing)
sudo puppet ssl bootstrap

# Show current certificate info
sudo puppet ssl show

# Verify the certificate chain
sudo puppet ssl verify

# Clean local SSL data (for re-registration)
sudo puppet ssl clean

# Submit a new CSR
sudo puppet ssl submit_request
```

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
