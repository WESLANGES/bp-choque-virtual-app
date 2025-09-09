# Plano de Trabalho - BPChoqueVirtual Git

## 1. Detalhes da Migra├º├úo e Estrutura Atual

Esta nova vers├úo ser├í implementada inicialmente em uma **base duplicada da vers├úo 1.0** atualmente em produ├º├úo, a qual foi desenvolvida em **Flutter**.

### Objetivo
Realizar o **m├¡nimo de altera├º├Áes poss├¡veis** na estrutura da base de dados.

### Diretrizes
- Qualquer altera├º├úo na base deve considerar impactos em m├║ltiplos pontos: **compatibilidade, integridade dos dados, performance e depend├¬ncias** do sistema legado.  
- Altera├º├Áes inevit├íveis devem ser **estudadas, analisadas e discutidas com a equipe** antes da aplica├º├úo.  
- Toda altera├º├úo dever├í ser **documentada**, acompanhada de um **plano de migra├º├úo** claro, para que o novo front-end possa ser ligado ├á base de dados 1.0 sem riscos de inconsist├¬ncia.  

ÔÜá´©Å Ao finalizarmos uma vers├úo m├¡nima, o pr├│ximo passo ser├í integrar o **novo front-end React** diretamente na **base de dados 1.0**, substituindo gradualmente o sistema atual em Flutter.

---

Foi realizada a **migra├º├úo completa** do c├│digo-fonte gerado pela plataforma **Lovable.dev** (`bpchoquevirtual-lovable`) para um reposit├│rio Git independente (`bpchoquevirtual-app`).  
O objetivo principal foi dar **maior autonomia ├á equipe t├®cnica**, al├®m de aumentar a **robustez no controle de vers├Áes**.

### Principais objetivos da migra├º├úo
- Garantir **controle total** sobre o c├│digo do front-end.  
- Possibilitar **edi├º├Áes manuais** e integra├º├Áes com IA (ex.: ChatGPT).  
- Obter **estabilidade e previsibilidade** na vers├úo em produ├º├úo.  
- Permitir **integra├º├úo e implanta├º├úo cont├¡nua (CI/CD)**, testes automatizados e documenta├º├úo.  

### Agora ├® poss├¡vel
- Adicionar/modificar p├íginas, componentes e estilos livremente.  
- Integrar gera├º├úo de c├│digo com IA.  
- Manter fluxo claro de versionamento (**main, dev, release**).  
- Garantir estabilidade em produ├º├úo com **testes locais antes da publica├º├úo**.  

---

## 2. Plano de Trabalho ÔÇô Reposit├│rios

- **bpchoquevirtual-lovable** ÔåÆ gera├º├úo de telas (interface visual via Lovable).  
- **bpchoquevirtual-app** ÔåÆ desenvolvimento da l├│gica funcional, integra├º├úo e ajustes.  

---

## 3. Fluxo de Trabalho para Nova Funcionalidade

### Exemplo: criar a funcionalidade de notifica├º├Áes

**3.1 No bpchoquevirtual-lovable (Dev 1):**  
- Criar telas no Lovable.  
- Exportar c├│digo para o `bpchoquevirtual-app`.  
- Comunicar a equipe que a base est├í pronta.  

**3.2 No bpchoquevirtual-app (Devs 2, 3 ou 4):**  
- Criar branch `feature/funcionalidade-notificacoes`.  
- Desenvolver e testar localmente.  
- Atualizar branch com `git pull origin main`.  
- Fazer commit e push.  

**3.3 Ap├│s testes e aprova├º├úo:**  
- Fazer merge com a `main`.  
- Resolver conflitos (se houver) com Meld.  
- Push para `origin main`.  

---

## 4. Integra├º├úo com a main
```bash
git checkout main
git pull origin main
git merge feature/funcionalidade
git push origin main
```

---

## 5. Boas pr├íticas
- Cada funcionalidade deve ter sua pr├│pria branch.  
- Commits com mensagens padronizadas (`feat`, `fix`, `refactor`, `docs`).  
- Sempre atualizar branch com `git pull` antes de come├ºar.  
- Deletar branches antigas ap├│s merge.  
- Testar e revisar antes do merge final.  

---

## 6. Observa├º├Áes T├®cnicas

Foi implementado o **custom_access_token_hook**, que personaliza o JWT automaticamente durante o login.  
Essa personaliza├º├úo adiciona informa├º├Áes espec├¡ficas do policial (**matr├¡cula, identifica├º├úo, unidade, etc.**) usadas para **controle multi-tenant**.

### Claims customizados
**1. Informa├º├Áes de Identifica├º├úo**
- `matricula` ÔåÆ matr├¡cula do policial  
- `identificacao` ÔåÆ identifica├º├úo do usu├írio  
- `id_usuario` ÔåÆ ID interno do usu├írio  

**2. Informa├º├Áes de Unidade (Multi-tenant)**
- `id_unidade` ÔåÆ ID da unidade (tenant)  
- `id_unidade_atual` ÔåÆ unidade atual de lota├º├úo  
- `sigla_unidade` ÔåÆ sigla da unidade  

**3. Estrutura do JWT**
```json
{
  "sub": "user_id",
  "email": "email@example.com",
  "app_metadata": {
    "claims": {
      "id_unidade": "uuid_da_unidade"
    }
  },
  "user_metadata": {
    "id_unidade": "uuid_da_unidade",
    "id_unidade_atual": "uuid_unidade_atual",
    "id_usuario": "id_interno_usuario",
    "identificacao": "identificacao_policial",
    "matricula": "matricula_policial",
    "sigla_unidade": "sigla_da_unidade"
  }
}
```

### Outras observa├º├Áes
- O metadata ├® atualizado via trigger `trg_atualizar_identificacao_user_metadata`.  
- **Edge Functions configuradas**:  
  - `access-token` ÔåÆ manipula├º├úo do JWT  
  - `primeiro-acesso` ÔåÆ cadastro de senha inicial  
  - `sgpol-search` ÔåÆ busca de policiais e fotos no SGPol  
- O contexto do usu├írio autenticado inclui:  
  `uid`, `id_policiais`, `id_unidade`, `id_unidade_atual`, `nomenclatura da unidade`, `c├│digo SGPol`.  

---

## 7. PERMISS├òES - Pilares A.V.O (Acesso, Visualiza├º├úo e Opera├º├úo)

A camada **A.V.O** organiza como o usu├írio interage com o sistema.  
Ela ├® composta por tr├¬s pilares complementares:  
- **Acesso** ÔåÆ m├│dulos e p├íginas que podem ser acessados (menus, cards, formul├írios).  
- **Visualiza├º├úo** ÔåÆ quais dados podem ser vistos (ex.: CPF, telefones).  
- **Opera├º├úo** ÔåÆ opera├º├Áes poss├¡veis (CRUD + notificar).  

### 7.1 Acesso
Tabelas envolvidas:
- `grupos` ÔåÆ m├│dulos principais  
- `subgrupos` ÔåÆ funcionalidades espec├¡ficas  
- `usuarios_roles` ÔåÆ conecta usu├írios a grupos/subgrupos  
- `permissoes` ÔåÆ hierarquia de opera├º├Áes  
- `view_usuarios_roles_ativos` ÔåÆ filtra apenas roles v├ílidos  

### 7.2 Visualiza├º├úo
Define quais campos de dados podem ser exibidos.  
**Tabela**: `campos_sensiveis`  
**Fun├º├úo**: `verificar_acesso_campo`  

Exemplo:
```sql
SELECT verificar_acesso_campo('view_policiais', 'cpf');
```

Permiss├Áes podem ser concedidas por:  
- usu├írio ÔåÆ `permissoes_usuario_campos_sensiveis`  
- grupo ÔåÆ `permissoes_grupo_campos_sensiveis`  

### 7.3 Opera├º├úo
Permiss├Áes hier├írquicas cumulativas:  
1. `ler` ÔåÆ visualizar  
2. `atualizar` ÔåÆ ler + editar  
3. `escrever` ÔåÆ atualizar + criar  
4. `deletar` ÔåÆ escrever + excluir  
5. `notificar` ÔåÆ escrever + notificar  

Exemplo:
```sql
SELECT p.nome AS permissao
FROM subgrupos s
JOIN permissoes p ON s.id_permissao = p.id
WHERE s.nome = 'Controle de Acesso';
```

### 7.4 Temporariedade de Acesso
**Tabela**: `usuarios_roles`  
Campos: `is_temporary`, `valid_from`, `valid_until`  
**View**: `view_usuarios_roles_ativos`  

Exemplo:
```sql
SELECT * FROM usuarios_roles
WHERE user_id = 'uuid_usuario'
AND is_temporary = true;
```
### 7.5 Fun├º├Áes de Verifica├º├úo de Permiss├Áes
- `verificar_minha_operacao(moduleId)`: Retorna o n├¡vel de opera├º├úo do usu├írio autenticado para um m├│dulo espec├¡fico. Consulta usuarios_credenciais filtrando pelo auth.uid() atual e retorna o id_permissoes associado ao m├│dulo.

- `verificar_operacao_usuario(userId, moduleId)`: Similar ├á anterior, mas verifica as opera├º├Áes de um usu├írio espec├¡fico (n├úo o autenticado). ├Ütil para administradores verificarem permiss├Áes de outros usu├írios.

- `verificar_meu_acesso(moduleId)`: Verifica se o usu├írio autenticado tem acesso a um m├│dulo. Retorna boolean consultando view_usuarios_roles_ativos com base no auth.uid() e filtrando por m├│dulo e temporalidade das roles.

- `verificar_acesso_usuario(userId, moduleId)`: Verifica acesso de um usu├írio espec├¡fico a um m├│dulo. Mesma l├│gica da anterior, mas para qualquer usu├írio (n├úo apenas o autenticado).

As fun├º├Áes "minha/meu" usam auth.uid() (contexto do usu├írio autenticado), enquanto as "usuario" recebem userId como par├ómetro (verifica├º├úo de terceiros).
---

## 8. Situa├º├úo Atual dos M├│dulos

| M├│dulo         | Status                   | Observa├º├Áes |
|----------------|-------------------------|-------------|
| Administra├º├úo  | Parcialmente implementado | - |
| Autentica├º├úo   | Funcional                | - |
| P├ígina Teste 2 | Implementado             | Criado com objetivo de validar o m├│dulo de permiss├Áes (A.V.O) |
| Layout/Design  | Migrado                  | Fontes e cores j├í migradas; layout pode ser adaptado conforme a necessidade |
