## Comfigurar/Atualizar o SSO(Single Sign-On) do provedor (idP) GSuite via SAML para o Microsoft Office 365.

Partindo do princípio de que você já utiliza o Google Workspace e deseja provisionar esses usuários no Microsoft Office 365, implementando o Single Sign-On (SSO). Dessa forma, os usuários poderão acessar ambas as plataformas com as mesmas credenciais, facilitando o gerenciamento e o acesso.

#### Caso deseje somente atualizar o certificado que venceu vá para o passo : 5
#### Antes de começar
- Adicione seu domínio do GSuite ao Microsoft Office 365. Mais instruções [aqui](https://learn.microsoft.com/pt-br/microsoft-365/admin/setup/add-domain?view=o365-worldwide).
- Faça login usando uma conta com privilégios de superadministrador. 

___
#### Primeiro adicione o Microsoft Office 365 no GSuite
1. No Google Admin Console, acesse **Menu > Apps > Apps da Web e para dispositivos móveis**.
2. Clique em **Adicionar app** e depois em **Pesquisar apps**.
3. Pesquise por **Office 365** e selecione o app **SAML Office 365**.
4. Na página de detalhes do provedor de identidade do Google, clique em **Continuar**.
   - Marque **Resposta assinada**.
   - Defina o **Formato do código de nome** como "PERSISTENTE".
5. No **Mapeamento de atributos**, mapeie os seguintes atributos:
   - **Primary Email** (Google) para **IDPEmail** (Office 365).
6. (Opcional) No campo de associação ao grupo, pesquise e selecione até 75 grupos conforme necessário.
