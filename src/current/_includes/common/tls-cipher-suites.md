{% if include.list == 'enabled' %}
- `TLS_DHE_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_DHE_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_DHE_RSA_WITH_AES_128_CCM`
- `TLS_DHE_RSA_WITH_AES_256_CCM`
- `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256`
- `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256`
- `TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256`
- `TLS_DHE_PSK_WITH_AES_128_GCM_SHA256`
- `TLS_DHE_PSK_WITH_AES_256_GCM_SHA384`
- `TLS_DHE_PSK_WITH_AES_128_CCM`
- `TLS_DHE_PSK_WITH_AES_256_CCM`
- `TLS_ECDHE_PSK_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_PSK_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_PSK_WITH_AES_128_CCM_SHA256`
- `TLS_ECDHE_PSK_WITH_CHACHA20_POLY1305_SHA256`
- `TLS_DHE_PSK_WITH_CHACHA20_POLY1305_SHA256`
{% endif %}
{% if include.list == 'disabled' %}
- `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`
- `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA`
- `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`
- `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA`
- `TLS_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_RSA_WITH_AES_128_CBC_SHA`
- `TLS_RSA_WITH_AES_256_CBC_SHA`
{% endif %}
