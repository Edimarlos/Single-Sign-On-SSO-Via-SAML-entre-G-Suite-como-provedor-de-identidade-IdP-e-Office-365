## Configurar/Atualizar o SSO(Single Sign-On) do provedor (idP) GSuite via SAML para o Microsoft Office 365.

Partindo do princípio de que você já utiliza o Google Workspace e deseja provisionar esses usuários no Microsoft Office 365, implementando o Single Sign-On (SSO). Dessa forma, os usuários poderão acessar ambas as plataformas com as mesmas credenciais, facilitando o gerenciamento e o acesso.

#### Caso deseje somente atualizar o certificado que venceu vá para o passo : 5
#### Antes de começar
- Adicione seu domínio do GSuite ao Microsoft Office 365. Mais instruções [aqui](https://learn.microsoft.com/pt-br/microsoft-365/admin/setup/add-domain?view=o365-worldwide).
- Faça login not GSuite usando uma conta com privilégios de superadministrador. 

___
#### Primeiro adicione o Microsoft Office 365 no GSuite
1. No Google Admin Console, acesse **Menu > Apps > Apps da Web e para dispositivos móveis**.
2. Clique em **Adicionar app** e depois em **Pesquisar apps**.
3. Pesquise por **Office 365** e selecione o app **SAML Office 365**.
4. Na página de detalhes do provedor de identidade do Google, clique em **Continuar**.
   - Marque **Resposta assinada**.
   - Defina o **Formato do código de nome** como "PERSISTENTE".

<img src="/assets/imgs/configGSuiteOffice365.png">

5. No **Mapeamento de atributos**, mapeie os seguintes atributos:
   - **Primary Email** (Google) para **IDPEmail** (Office 365).

Mais instruções [aqui](https://support.google.com/a/answer/6363817?hl=pt-BR&sjid=1860868631779308443-SA#zippy=%2Cstep-configure-immutableid%2Cstep-set-up-google-as-a-saml-identity-provider-idp%2Cantes-de-come%C3%A7ar%2Cetapa-configurar-o-immutableid%2Cetapa-receber-informa%C3%A7%C3%B5es-do-provedor-de-identidade-idp-do-google%2Cetapa-configurar-o-google-como-provedor-de-identidade-idp-saml).

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


Mais instruções [aqui](https://support.google.com/a/answer/7365072?sjid=1860868631779308443-SA#zippy=%2Cconfigurar-o-provisionamento-autom%C3%A1tico-para-o-aplicativo-microsoft-office).

___
####  Configurar o Office 365 com os parâmetros/certificado do GSuite
```
   [xml]$idp = Get-Content C:\Path\to\xml\GoogleIDPMetadata-$domainName.xml

   $activeLogonUri = "https://login.microsoftonline.com/login.srf"
   $signingCertificate = ($idp.EntityDescriptor.IDPSSODescriptor.KeyDescriptor.KeyInfo.X509Data.X509Certificate | Out-String).Trim()
   $issuerUri = $idp.EntityDescriptor.entityID
   $logOffUri = $idp.EntityDescriptor.IDPSSODescriptor.SingleSignOnService.Location[0]
   $passiveLogOnUri = $idp.EntityDescriptor.IDPSSODescriptor.SingleSignOnService.Location[0]

   Set-MsolDomainAuthentication `
   -DomainName $domainName `
   -FederationBrandName $domainName `
   -Authentication Federated `
   -PassiveLogOnUri $passiveLogOnUri `
   -ActiveLogOnUri $activeLogonUri `
   -SigningCertificate $signingCertificate `
   -IssuerUri $issuerUri `
   -LogOffUri $logOffUri `
   -PreferredAuthenticationProtocol "SAMLP"
```