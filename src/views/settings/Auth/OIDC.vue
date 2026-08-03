<template>
  <BaseAuth :config="settings" enable-field="AUTH_OPENID">
    <div class="callback-url">
      <span class="callback-url-label">{{ $t('OIDCCallbackURL') }}</span>
      <el-input :model-value="callbackUrl" readonly>
        <template #append>
          <el-tooltip :content="$t('Copy')" placement="top">
            <el-button @click="copyCallbackUrl">
              <el-icon><CopyDocument /></el-icon>
            </el-button>
          </el-tooltip>
        </template>
      </el-input>
    </div>
  </BaseAuth>
</template>

<script>
import BaseAuth from './Base'
import { JsonEditor, UpdateToken } from '@/components/Form/FormFields'
import { JsonRequired } from '@/components/Form/DataForm/rules'
import { getOrgSelect2Meta } from '@/views/settings/Auth/const'
import { copy } from '@/utils/common/index'
export default {
  name: 'OIDC',
  components: {
    BaseAuth
  },
  data() {
    return {
      settings: {
        url: '/api/v1/settings/setting/?category=oidc',
        encryptedFields: ['AUTH_OPENID_CLIENT_SECRET'],
        fields: [
          [
            this.$t('Basic'),
            [
              'AUTH_OPENID',
              'BASE_SITE_URL',
              'AUTH_OPENID_CLIENT_ID',
              'AUTH_OPENID_CLIENT_SECRET',
              'AUTH_OPENID_CLIENT_AUTH_METHOD'
            ]
          ],
          [
            this.$t('Server'),
            [
              'AUTH_OPENID_KEYCLOAK',
              'AUTH_OPENID_SERVER_URL',
              'AUTH_OPENID_REALM_NAME',
              'AUTH_OPENID_PROVIDER_ENDPOINT',
              'AUTH_OPENID_PROVIDER_AUTHORIZATION_ENDPOINT',
              'AUTH_OPENID_PROVIDER_TOKEN_ENDPOINT',
              'AUTH_OPENID_PROVIDER_JWKS_ENDPOINT',
              'AUTH_OPENID_PROVIDER_USERINFO_ENDPOINT',
              'AUTH_OPENID_PROVIDER_END_SESSION_ENDPOINT',
              'AUTH_OPENID_PROVIDER_SIGNATURE_ALG',
              'AUTH_OPENID_PROVIDER_SIGNATURE_KEY',
              'AUTH_OPENID_PKCE',
              'AUTH_OPENID_CODE_CHALLENGE_METHOD',
              'AUTH_OPENID_SCOPES',
              'AUTH_OPENID_ID_TOKEN_MAX_AGE',
              'AUTH_OPENID_ID_TOKEN_INCLUDE_CLAIMS',
              'AUTH_OPENID_USE_STATE',
              'AUTH_OPENID_USE_NONCE',
              'AUTH_OPENID_ALWAYS_UPDATE_USER',
              'AUTH_OPENID_IGNORE_SSL_VERIFICATION',
              'AUTH_OPENID_SHARE_SESSION'
            ]
          ],
          [this.$t('Search'), ['AUTH_OPENID_USER_ATTR_MAP']],
          [this.$t('Other'), ['OPENID_ORG_IDS']]
        ],
        fieldsMeta: {
          AUTH_OPENID_CLIENT_SECRET: {
            component: UpdateToken
          },
          AUTH_OPENID_SERVER_URL: {
            hidden: (form) => !form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_REALM_NAME: {
            hidden: (form) => !form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_ENDPOINT: {
            helpTextAsTip: false,
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_AUTHORIZATION_ENDPOINT: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_TOKEN_ENDPOINT: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_JWKS_ENDPOINT: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_USERINFO_ENDPOINT: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_END_SESSION_ENDPOINT: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_SIGNATURE_ALG: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PROVIDER_SIGNATURE_KEY: {
            hidden: (form) => form['AUTH_OPENID_KEYCLOAK']
          },
          AUTH_OPENID_PKCE: {
            hidden: false
          },
          AUTH_OPENID_CODE_CHALLENGE_METHOD: {
            hidden: (form) => !form['AUTH_OPENID_PKCE']
          },
          AUTH_OPENID_SCOPES: {
            hidden: false
          },
          AUTH_OPENID_ID_TOKEN_MAX_AGE: {
            hidden: false
          },
          AUTH_OPENID_ID_TOKEN_INCLUDE_CLAIMS: {
            hidden: false
          },
          AUTH_OPENID_USE_STATE: {
            hidden: false
          },
          AUTH_OPENID_USE_NONCE: {
            hidden: false
          },
          AUTH_OPENID_IGNORE_SSL_VERIFICATION: {},
          AUTH_OPENID_SHARE_SESSION: {},
          AUTH_OPENID_USER_ATTR_MAP: {
            component: JsonEditor,
            rules: [JsonRequired]
          },
          OPENID_ORG_IDS: getOrgSelect2Meta({ licenseRequired: false })
        },
        moreButtons: [
          {
            title: this.$t('Test'),
            loading: false,
            callback: (value, form, button) => {
              button.loading = true
              const discoveryConfig = {
                AUTH_OPENID_KEYCLOAK: value.AUTH_OPENID_KEYCLOAK,
                AUTH_OPENID_SERVER_URL: value.AUTH_OPENID_SERVER_URL,
                AUTH_OPENID_REALM_NAME: value.AUTH_OPENID_REALM_NAME,
                AUTH_OPENID_PROVIDER_ENDPOINT: value.AUTH_OPENID_PROVIDER_ENDPOINT,
                AUTH_OPENID_IGNORE_SSL_VERIFICATION: value.AUTH_OPENID_IGNORE_SSL_VERIFICATION,
                BASE_SITE_URL: value.BASE_SITE_URL
              }
              this.$axios
                .post('/api/v1/settings/oidc/testing/', discoveryConfig, {
                  disableFlashErrorMsg: true
                })
                .then((res) => {
                  const discovered = {
                    AUTH_OPENID_PROVIDER_ENDPOINT: res.issuer,
                    ...(res.endpoints || {})
                  }
                  form.updateForm(discovered)
                  this.testedCallbackUrl = res.callback_url
                  this.$message.success(res.msg)
                })
                .catch((error) => {
                  const data = error.response?.data || {}
                  const validationErrors = Object.values(data).flat().join('; ')
                  const message = data.error || validationErrors || error.message
                  this.$message.error(message)
                })
                .finally(() => {
                  button.loading = false
                })
            }
          }
        ],
        submitMethod: () => 'patch',
        afterGetFormValue(obj) {
          return obj
        },
        cleanFormValue(data) {
          if (data['AUTH_OPENID_CLIENT_SECRET'] === '') {
            delete data['AUTH_OPENID_CLIENT_SECRET']
          }
          return data
        }
      },
      testedCallbackUrl: ''
    }
  },
  computed: {
    callbackUrl() {
      return this.testedCallbackUrl || `${window.location.origin}/core/auth/openid/callback/`
    }
  },
  methods: {
    copyCallbackUrl() {
      copy(this.callbackUrl)
    }
  }
}
</script>

<style lang="scss" scoped>
.callback-url {
  display: flex;
  align-items: center;
  gap: 18px;
  padding: 20px 0 0;

  .callback-url-label {
    flex: 0 0 18.2%;
    text-align: right;
    font-size: 13px;
    color: var(--color-text-primary);
  }

  .el-input {
    max-width: 720px;
  }
}

@media (max-width: 768px) {
  .callback-url {
    align-items: stretch;
    flex-direction: column;
    gap: 8px;

    .callback-url-label {
      flex: 0 0 auto;
      text-align: left;
    }
  }
}
</style>
