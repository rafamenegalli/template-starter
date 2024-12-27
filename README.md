# BRASMEG RH
* Documentação referente a Importação de Horas
* Cálculos para as Bases Mauá e Cubatão

https://blush-olive-fc1.notion.site/IMPORTA-O-AUTO-TRAC-463a420ea7d846a09a0115838268a083

# BRASMEG V2
## Nova branch brasmegV2

- **Implementação no Checklist**:
  - Captura de data e hora do início do preenchimento do questionário;
  - Captura de data e hora do envio do checklist;
  - Acesso ao checklist com CPF e Senha;
  - Captura de Geolocalização ao iniciar e finalizar o checklist;
  - Integração de Geolocalização com API de recuperação de endereço.

- **Cadastro de Usuários**:
  - Redefinição de Senha;
  - Permissão para acesso ao Sistema Principal.

---

## Resumo

- O usuário do checklist precisará fornecer uma senha para acessar o sistema.
- A regra para formação da primeira senha será:
  - Os 9 primeiros dígitos do CPF (exemplo: 123.456.789 → a senha será: 123456789).

- Acessando o checklist pela primeira vez, será obrigatória a troca de senha. A senha deverá ter no mínimo 6 caracteres, e a nova senha não poderá ser igual à primeira senha, ou seja, os dígitos do CPF na mesma sequência.

- Com esta senha, o usuário poderá acessar apenas o checklist. O sistema principal não deve ser acessado por ele.

- Ao iniciar o questionário, serão salvos a data, hora e geolocalização. Ao enviar o questionário, as informações serão armazenadas, tendo dados iniciais e finais. Porém, em tela, para o usuário do checklist, estas informações não estarão disponíveis. Elas irão aparecer apenas no sistema principal. Não foi implementado para o cliente a visualização de DIVERGÊNCIA DE GEOLOCALIZAÇÃO. Porém, como prevenção para uma auditoria, estes dados já estão sendo armazenados.

- O usuário do Checklist não terá a opção de "Esqueci minha senha". Caso seja necessário redefinir a senha, a ação poderá ser executada pelos usuários do sistema que contenham a permissão de "Administrador do Sistema". Será exibido um botão "Redefinir Senha". Esta opção está disponível no Cadastro de Usuários/Colaboradores e Cadastro de Usuários/Terceiros (no módulo checklist).

- Para redefinição de senha será obrigatória uma justificativa para redefinir a senha.

- Caso a senha do usuário tenha sido redefinida, ao acessar o checklist, será obrigatório o cadastro de uma nova senha.

- Na tela de cadastro de usuários, com permissão ADMINISTRATIVA, foi adicionada uma opção que permitirá definir se aquele CPF poderá acessar o sistema principal.

---

## Resumo Técnico

- Foi criada uma flag **"Acessa Sistema Principal"**, que apenas os usuários com regra de ADMINISTRAÇÃO poderão ativar ou desativar. A mesma regra vale para redefinição de senha.

- Foi necessário alterar diversas partes do sistema para evitar burlar estas travas, alterando os métodos de login e "esqueci a senha" (que não estará disponível no checklist). A validação da coluna **"user_access_system_brasmeg"** só permitirá acesso ao sistema principal se estiver como TRUE.

- Foi criada a tabela **"control_settings"** para controlar os acessos. Esta tabela será responsável por armazenar dados referentes ao usuário e acesso.

- O login realizado no checklist irá validar em paralelo com esta tabela, para verificar se é primeiro acesso ou redefinição de senha.

- Também foi criada uma tabela para armazenar as justificativas (**"password_justifications"**) para as redefinições de senha. Sempre que for solicitado, será armazenada a justificativa nesta tabela, junto com o ID do Usuário e um resumo de quem está solicitando a redefinição e a data/hora da solicitação.

- Estes dados estão disponíveis apenas para consulta direta em banco de dados, para futuras auditorias.

  - (Foi separada em duas tabelas, para evitar uma tabela `users` com muitas colunas. Todas as tabelas criadas estão com **constraint** na chave primária `user_id`).

- No FrontEnd, todo escopo de "Checklist" foi alterado para atender os requisitos solicitados, exceto a página referente à busca de veículos e checklists já enviados pelo usuário. O input de senha só será exibido após consultar o CPF. Se for primeiro acesso ou redefinição de senha, será redirecionado para a tela de "REDEFINIÇÃO DE SENHA OBRIGATÓRIA".

- Ao iniciar o questionário serão capturados:
  - Data, Hora e Geolocalização (Latitude e Longitude), sendo armazenados como **_init** e **_start**.

- Ao enviar o questionário serão capturados novamente:
  - Data, Hora e Geolocalização, sendo armazenados como **_finish**.

- A Geolocalização é obtida através do navegador web do usuário, que é baseada na localização do IP. Isso pode gerar uma área de precisão incorreta. A API que está sendo utilizada para busca de endereços é a **OpenStreet**, que é openSource e utilizada através de um encurtador chamado **Nominatim**, sem necessidade de criar um TOKEN de acesso.

- Foi implementada a opção de busca de endereços também pela API do **Google Maps**, caso o cliente queira maior precisão. Porém, para a API do Google, serão gerados custos. A definição de qual API o backend está utilizando é por parâmetro global. Vide: **ParametersGlobals**, `useApiGoogle`. Se estiver como false, utilizará como padrão a OpenStreet.

- Para armazenar esses dados, foram adicionadas colunas na tabela **filled_checklists**.

---

## IMPORTANTE

- **PARA ATUALIZAR EM PRODUÇÃO, VERIFICAR OS ARQUIVOS DE SCRIPTS**, pois será necessário criar tabelas e executar alterações de colunas em tabelas diversas. No arquivo de script está descrito.

---

## Notas de Melhoria

- Hoje não existe nenhum tratamento para dificultar a geração de senha de usuários. Isso permite que usuários diferentes possuam a mesma senha.  
  Exemplo:
  - CPF: `000.000.000-00` → Senha: `1234`
  - CPF: `111.111.111-11` → Senha: `1234`

- Isso abre precedentes para que um usuário mal-intencionado utilize a senha para acessar outros CPFs. Pode ser implementada uma lógica para dificultar, criando uma alternativa que, mesmo tendo a mesma senha, garanta que esta senha pertença ao usuário correto.
#   t e m p l a t e - s t a r t e r  
 