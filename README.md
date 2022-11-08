gcloud auth login admin-krissak@gcp.devopsinuse.com
gcloud config set project wif-devopsinuse-tfe
GCP_CREDS=~/.config/gcloud/legacy_credentials/admin-krissak@gcp.devopsinuse.com/adc.json
vault secrets enable gcp
vault write gcp/config credentials=@${GCP_CREDS} ttl=7200 max_ttl=28800
vault read gcp/config
vault write gcp/roleset/myrole secret_type="access_token" project="wif-devopsinuse-tfe" bindings=@bindings.hcl token_scopes="https://www.googleapis.com/auth/cloud-platform"
curl -H "X-Vault-Token: $VAULT_TOKEN" -XGET $VAULT_ADDR/v1/gcp/token/myrole > out.json
TOKEN="$(jq -r .data.token out.json)"
curl -H "Authorization: Bearer $TOKEN" https://storage.googleapis.com/storage/v1/b/wif-devopsinuse-tfe-tfe-oidc-test-bucket

vault write gcp/static-account/my-key-account \
  service_account_email="sa-tfe@wif-devopsinuse-tfe.iam.gserviceaccount.com" \
  secret_type="service_account_key"

vault read gcp/static-account/my-key-account/key
