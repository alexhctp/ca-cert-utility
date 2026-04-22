# ca-cert-utility.sh

Bash script que extrai a cadeia de certificados TLS de um servidor remoto e instala os certificados no trust store do sistema Linux.

## O que o script faz

1. **Extrai a cadeia de certificados** do servidor `wikipedia.org:443` usando `openssl s_client` com verificação até 5 níveis de profundidade.
2. **Divide os certificados** individuais em arquivos `.pem` separados (`cert1.pem`, `cert2.pem`, ...) via `awk`.
3. **Renomeia cada certificado** com base no campo `CN` (Common Name) do subject, normalizando o nome:
   - Converte para letras minúsculas
   - Substitui espaços, vírgulas e asteriscos por `_`
   - Remove underscores duplicados e prefixos desnecessários
   - Produz arquivos `.crt` com nomes legíveis (ex: `digicert_global_root_ca.crt`)
4. **Move os arquivos `.crt`** para `/usr/local/share/ca-certificates` (requer `sudo`).
5. **Atualiza o trust store** do sistema com `sudo update-ca-certificates`.

## Uso

```bash
chmod +x ca-cert-utility.sh
./ca-cert-utility.sh
```

> O script deve ser executado em um diretório com permissão de escrita. Requer `sudo` para as etapas de instalação.

## Dependências

| Ferramenta | Finalidade |
|---|---|
| `openssl` | Conexão TLS e leitura de certificados |
| `awk` | Divisão da cadeia de certificados em arquivos |
| `sed` / `tr` | Normalização dos nomes de arquivo |
| `sudo` | Escrita em `/usr/local/share/ca-certificates` |
| `update-ca-certificates` | Atualização do trust store (Debian/Ubuntu) |

## Casos de uso práticos

- **Ambientes corporativos com proxy TLS/SSL inspection** — instala o certificado intermediário do proxy para evitar erros de SSL.
- **Pipelines CI/CD** — corrige falhas de build causadas por certificados não confiáveis ao conectar a repositórios (Artifactory, Nexus, GitLab).
- **Onboarding de novos servidores Linux** — automatiza a instalação de CAs customizadas em múltiplas máquinas.
- **Integração com serviços internos** — necessário quando Artifactory, Jenkins, ou outros serviços usam certificados de CA privada.
- **Troubleshooting de handshake TLS** — permite inspecionar e instalar manualmente a cadeia de certificados de qualquer host.
- **Ambientes isolados / air-gapped** — propaga CAs internas sem acesso a repositórios públicos.
- **Migração de certificados** — útil ao trocar CA ou intermediário, renovando o trust store sem reconfigurar cada aplicação.

## Customização

Para usar com um host diferente de `wikipedia.org`, altere a linha:

```bash
openssl s_client -showcerts -verify 5 -connect wikipedia.org:443 < /dev/null |
```

Substitua `wikipedia.org:443` pelo endereço desejado, por exemplo:

```bash
openssl s_client -showcerts -verify 5 -connect artifactory.example.com:443 < /dev/null |
```

## Observações

- Compatível com distribuições **Debian/Ubuntu** (usa `update-ca-certificates`). Para RHEL/CentOS, substituir por `update-ca-trust`.
- Os arquivos `.pem` e `.crt` são gerados no diretório corrente — recomenda-se executar em um diretório temporário.
- O script não valida se os certificados já estão instalados; execuções repetidas sobrescrevem os arquivos anteriores.
