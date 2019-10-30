# powershell_vault_access
Accessing vault via powershell

# Userdata to install vault

```bash
<powershell>
$securityProtocolSettingsOriginal = [System.Net.ServicePointManager]::SecurityProtocol

try {
  [System.Net.ServicePointManager]::SecurityProtocol = 3072 -bor 768 -bor 192 -bor 48
} catch {
  Write-Warning 'Unable to set PowerShell to use TLS 1.2 and TLS 1.1 due to old .NET Framework installed. If you see underlying connection closed or trust errors, you may need to do one or more of the following: (1) upgrade to .NET Framework 4.5 and PowerShell v3, (2) specify internal Chocolatey package location (set $env:chocolateyDownloadUrl prior to install or host the package internally), (3) use the Download + PowerShell method of install. See https://chocolatey.org/install for all install options.'
}

iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

[System.Net.ServicePointManager]::SecurityProtocol = $securityProtocolSettingsOriginal
choco install vault -y
</powershell>
```

# Vault prerequisites

```bash

vault auth enable aws

vault kv put secret/foo bar=baz
 
vault policy write "example-policy" -<<EOF
path "secret/foo*" {
  capabilities = ["create", "read"]
}
path "secret/" {
  capabilities = ["list", "read"]
}
EOF

vault write auth/aws/role/MyRole auth_type=iam policies=example-policy max_ttl=500h bound_iam_principal_arn=arn:aws:iam::99999999:role/MyRole inferred_aws_region="us-east-1" inferred_entity_type="ec2_instance"


```


# Powershell to access vault

```bash
$env:VAULT_ADDR="http://x.x.x.x:8200"
$results=(vault login -method=aws)
$results_json = $results | ConvertFrom-Json
$token = $results_json.token
vault login $token
$data=(vault kv get -format=json secret/foo)
$json_data = $data | ConvertFrom-Json
$objMembers = $json_data.data.psobject.Members | where-object membertype -like 'noteproperty'
$form = @{}
foreach ($obj in $objMembers){
 $form.add("$($obj.name)","$($obj.Value)")
}
echo $form

Name                           Value
----                           -----
bar                            baz

echo $form.bar
baz
```

