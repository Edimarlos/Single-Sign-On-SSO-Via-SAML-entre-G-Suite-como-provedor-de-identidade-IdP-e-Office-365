## Configurar/Atualizar o SSO(Single Sign-On) do provedor (idP) Google Workspace via SAML para o Microsoft Office 365.

Partindo do princípio de que você já utiliza o Google Workspace e deseja provisionar esses usuários no Microsoft Office 365, implementando o Single Sign-On (SSO). Dessa forma, os usuários poderão acessar ambas as plataformas com as mesmas credenciais, facilitando o gerenciamento e o acesso.

> **Nota:** Caso deseje somente atualizar o certificado que venceu [clique aqui](#atualizando-certificado-quando-estiver-vencido).

#### Antes de começar
- Adicione seu domínio do Google Workspace ao Microsoft Office 365. Mais instruções [clique aqui](https://learn.microsoft.com/pt-br/microsoft-365/admin/setup/add-domain?view=o365-worldwide).
- Faça login not Google Workspace usando uma conta com privilégios de superadministrador. 

___
#### Primeiro adicione o Microsoft Office 365 no Google Workspace
1. No Google Admin Console, acesse **Menu > Apps > Apps da Web e para dispositivos móveis**.
2. Clique em **Adicionar app** e depois em **Pesquisar apps**.
3. Pesquise por **Office 365** e selecione o app **SAML Office 365**.
4. Na página de detalhes do provedor de identidade do Google, clique em **Continuar**.
   - Marque **Resposta assinada**.
   - Defina o **Formato do código de nome** como "PERSISTENTE".

<img src="/assets/imgs/configGSuiteOffice365.png">

5. No **Mapeamento de atributos**, mapeie os seguintes atributos:
   - **Primary Email** (Google) para **IDPEmail** (Office 365).

Mais instruções [clique aqui](https://support.google.com/a/answer/6363817?hl=pt-BR&sjid=1860868631779308443-SA#zippy=%2Cstep-configure-immutableid%2Cstep-set-up-google-as-a-saml-identity-provider-idp%2Cantes-de-come%C3%A7ar%2Cetapa-configurar-o-immutableid%2Cetapa-receber-informa%C3%A7%C3%B5es-do-provedor-de-identidade-idp-do-google%2Cetapa-configurar-o-google-como-provedor-de-identidade-idp-saml).

___

#### Configurar o provisionamento automático para Office 365

6. Na seção **Provisionamento automático**, clique em **Configurar provisionamento automático** e depois em **Autorizar**.
7. Faça login na sua conta de administrador do Office 365, se necessário.
   > **Nota:** Se a senha do administrador do Office 365 mudar, será necessário reautorizar.
8. Verifique se os atributos obrigatórios do Office 365 estão mapeados corretamente. Ajuste conforme necessário.

<img src="/assets/imgs/mapeamentoGSuiteOffice365.png">

9. (Opcional) Restrinja o provisionamento a grupos específicos pesquisando e adicionando grupos.
10. Defina o período de adiamento para desprovisionamento e as ações a serem tomadas (suspender ou excluir a conta) após o prazo estabelecido.
11. Ative o provisionamento automático

Mais instruções [clique aqui](https://support.google.com/a/answer/7365072?sjid=1860868631779308443-SA#zippy=%2Cconfigurar-o-provisionamento-autom%C3%A1tico-para-o-aplicativo-microsoft-office).

12. Baixe o certificado que contem os metadados necessários para o usar no Office 365 no **Menu > Apps > Apps da Web e para dispositivos móveis > Selecione o Office 365> Fazer dowbload do metadados**
<img src="/assets/imgs/baixarMetadados.png">
<img src="/assets/imgs/metadadosCertificadoGSuite.png">
___

####  Configurar o Office 365 com os parâmetros/certificado do Google Workspace

13. Excecute o PowerShell como Administrador.

14. Instale, importe e conecte usando usuário com perfil de administrador ao Office 365.
```
   Install-Module MSOnline
   Import-Module MSOnline
   Connect-MsolService
```
15. Crie as seguintes variáveis e insira o seu domínio e o local onde foi salvo o arquivo com os metadados do certificado baixado do Google Workspace.
```   
   $domainName = "<seu-dominio.com>"
   [xml]$idp = Get-Content <local do arquivo com os metadados XML>      

   $activeLogonUri = "https://login.microsoftonline.com/login.srf"
   $signingCertificate = ($idp.EntityDescriptor.IDPSSODescriptor.KeyDescriptor.KeyInfo.X509Data.X509Certificate | Out-String).Trim()
   $issuerUri = $idp.EntityDescriptor.entityID
   $logOffUri = $idp.EntityDescriptor.IDPSSODescriptor.SingleSignOnService.Location[0]
   $passiveLogOnUri = $idp.EntityDescriptor.IDPSSODescriptor.SingleSignOnService.Location[0]
```

16. Finalize definindo o domínio como federado.
```
   Set-MsolDomainAuthentication -DomainName $domainName -FederationBrandName $domainName -Authentication Federated -PassiveLogOnUri $passiveLogOnUri -ActiveLogOnUri $activeLogonUri -SigningCertificate $signingcertificate -IssuerUri $issuerUri -LogOffUri $logOffUri -PreferredAuthenticationProtocol "SAMLP"
```

17. Para testar, volte no Google Workspace, acesse: **Menu > Apps > Apps da Web e para dispositivos móveis > Selecione o Office 365 > Clique em Testar login SAML.**

___

## Atualizando Certificado Quando Estiver Vencido
> **Nota:** O certificado tem um vencimento, mas quando ele expirar, basta seguir os seguintes passos:

1. Faça login no Google Workspace usando uma conta com privilégios de superadministrador e acesse: **Menu > Apps > Apps da Web e para dispositivos móveis > Selecione o Office 365 > Detalhes do provedor de serviços > Certificado.** 

2. Selecione o novo certificado (caso necessário, gere um novo) e, após salvar, baixe o certificado que contém os metadados necessários para usar no Office 365.
<img src="/assets/imgs/atualizarCertificado.png">

3. Excecute o PowerShell como Administrador e rode os comandos dos passos 14 e 15 anteriores da configuração anterior. 

4. Agora finalize atualizando os dados
```   
   Set-MsolDomainFederationSettings -DomainName $domainName -FederationBrandName $domainName -PassiveLogOnUri $passiveLogOnUri -ActiveLogOnUri $activeLogonUri -SigningCertificate $signingcertificate -NextSigningCertificate $signingcertificate -IssuerUri $issuerUri -LogOffUri $logOffUri -PreferredAuthenticationProtocol "SAMLP"
```

####  Comandos úteis

Para verificar a autenticação atual do domínio, use:
```   
   Get-MsolDomainFederationSettings -DomainName "{seu-dominio.com}" | Format-List *
```

Listar Usuários do Domínio:
```   
   Get-MsolUser -DomainName "{seu-dominio.com}"
```

___

##### Referências:
###### https://medium.com/@james.winegar/how-to-single-sign-on-sso-between-g-suite-and-office-365-with-g-suite-as-identity-provider-idp-5bf5031835a0

###### https://support.google.com/a/answer/6363817?hl=pt-BR&sjid=1860868631779308443-SA#zippy=%2Cstep-configure-immutableid%2Cstep-set-up-google-as-a-saml-identity-provider-idp%2Cantes-de-come%C3%A7ar%2Cetapa-configurar-o-immutableid%2Cetapa-receber-informa%C3%A7%C3%B5es-do-provedor-de-identidade-idp-do-google%2Cetapa-configurar-o-google-como-provedor-de-identidade-idp-saml

###### https://github.com/IAmFrench/GSuite-as-identity-Provider-IdP-for-Office-365-or-Azure-Active-Directory