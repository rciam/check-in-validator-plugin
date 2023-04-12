# check-in-validator-plugin

A plugin for checking if an Access Token issued by EGI Check-in is valid. This
plugin can be used by HTCondor-CE and ARC-CE.

## Installation

This plugin can be installed by downloading the rpm packages from the releases
or by using the source code. The source code should be used only for
development.

### Release

You can find all the available rpm packages in the
[releases](https://github.com/rciam/check-in-validator-plugin/releases).

### Source code (For development)

To install the plugin using source code, run the following commands:

```bash
git clone https://github.com/rciam/check-in-validator-plugin.git
cd check-in-validator-plugin
python check-in-validator-plugin.py
```

## Configuration

This section is covering how to configure the plugin in HTCondor-CE.

### HTCondor

Set up an HTCondor-CE configuration as usual, then install/update the condor
packages provided here. No update to the HTCondor-CE packages is needed.
For the simplest configuration, add the following line to
`/etc/condor-ce/mapfiles.d/10-scitokens.conf` (this assumes there’s only one
issuer of EGI Check-in tokens):

```text
SCITOKENS /^https:\/\/aai-dev.egi.eu\/auth\/realms\/egi,.*/ PLUGIN:*
```

Then, create a file under `/etc/condor-ce/config.d/` like this:

```text
SEC_SCITOKENS_ALLOW_FOREIGN_TOKENS=true
SEC_SCITOKENS_PLUGIN_NAMES=EGI-CHECK-IN-VALIDATOR
SEC_SCITOKENS_PLUGIN_EGI-CHECK-IN-VALIDATOR_COMMAND=$(LOCAL_DIR)/egi-check-in-validator.plugin
SEC_SCITOKENS_PLUGIN_EGI-CHECK-IN-VALIDATOR_MAPPING=egi-check-in
```

## How the plugin works

The plugin is expecting as input the payload of the JWT as decoded json in
string format in order to validate the token. If the JWT will not be provided
via stdin within 5 seconds, then the plugin will use the environment variables
that HTCondor/ARC creates. After parsing the JWT, the plugin will create
environment variables for each group that the user in member of. The format of
the environment variables have the following format:

```text
BEARER_TOKEN_0_GROUP_*
```

### Example providing JWT via stdin

Example input:

```bash
$ python egi-check-in-validator.py
Insert JWT: 
{"exp":1681213287,"iat":1681209687,"auth_time":1681209570,"jti":"92cfba6e-7c6b-4012-9f6c-2539ef1b76f6","iss":"https://aai-dev.egi.eu/auth/realms/egi","sub":"bf009c87cb04f0a69fb2cc98767147e5b7408bedaef07b70ef33ef777318e610@egi.eu","typ":"Bearer","azp":"myClientID","nonce":"c2651c777c2c888fcf8244c22b1bcb14","session_state":"515679aa-b818-4902-ae7f-49b198aa0661","scope":"openid offline_access eduperson_entitlement voperson_id eduperson_entitlement_jwt eduperson_entitlement_jwt:urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu profile email","sid":"515679aa-b818-4902-ae7f-49b198aa0661","voperson_id":"bf009c87cb04f0a69fb2cc98767147e5b7408bedaef07b70ef33ef777318e610@egi.eu","authenticating_authority":"https://idp.admin.grnet.gr/idp/shibboleth","eduperson_entitlement":["urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu"]}
0
```

Example output:

```text
BEARER_TOKEN_0_GROUP_0=urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu
```

### Example providing JWT via environment variables

Example input:

```bash
$ export BEARER_TOKEN_0_CLAIM_voperson_id_0=bf009c87cb04f0a69fb2cc98767147e5b7408bedaef07b70ef33ef777318e610@egi.eu
$ export BEARER_TOKEN_0_CLAIM_eduperson_entitlement_0=urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu
$ export BEARER_TOKEN_0_CLAIM_eduperson_entitlement_1=urn:mace:egi.eu:group:vo.example.org:role=manager#aai.egi.eu
$ export BEARER_TOKEN_0_SCOPE_0=openid
$ export BEARER_TOKEN_0_SCOPE_1=compute.modify
$ export BEARER_TOKEN_0_SCOPE_2=compute.create
$ export BEARER_TOKEN_0_SCOPE_3=compute.read
$ export BEARER_TOKEN_0_SCOPE_4=eduperson_entitlement
$ export BEARER_TOKEN_0_SCOPE_5=voperson_id
$ export BEARER_TOKEN_0_SCOPE_6=profile
$ export BEARER_TOKEN_0_SCOPE_7=email
$ python egi-check-in-validator.py
Insert JWT:
0
```

Example output:

```text
BEARER_TOKEN_0_GROUP_0=urn:mace:egi.eu:group:vo.example.org:role=member#aai.egi.eu
BEARER_TOKEN_0_GROUP_1=urn:mace:egi.eu:group:vo.example.org:role=manager#aai.egi.eu
```
