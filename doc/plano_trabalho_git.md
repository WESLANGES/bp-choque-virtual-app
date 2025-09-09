# Plano de Trabalho - BPChoqueVirtual Git

## 1. Detalhes da Migração e Estrutura Atual

Esta nova versão será implementada inicialmente em uma **base duplicada da versão 1.0** atualmente em produção, a qual foi desenvolvida em **Flutter**.

### Objetivo
Realizar o **mínimo de alterações possíveis** na estrutura da base de dados.

### Diretrizes
- Qualquer alteração na base deve considerar impactos em múltiplos pontos: **compatibilidade, integridade dos dados, performance e dependências** do sistema legado.  
- Alterações inevitáveis devem ser **estudadas, analisadas e discutidas com a equipe** antes da aplicação.  
- Toda alteração deverá ser **documentada**, acompanhada de um **plano de migração** claro, para que o novo front-end possa ser ligado à base de dados 1.0 sem riscos de inconsistência.  

⚠️ Ao finalizarmos uma versão mínima, o próximo passo será integrar o **novo front-end React** diretamente na **base de dados 1.0**, substituindo gradualmente o sistema atual em Flutter.

---

Foi realizada a **migração completa** do código-fonte gerado pela plataforma **Lovable.dev** (`bpchoquevirtual-lovable`) para um repositório Git independente (`bpchoquevirtual-app`).  
O objetivo principal foi dar **maior autonomia à equipe técnica**, além de aumentar a **robustez no controle de versões**.

### Principais objetivos da migração
- Garantir **controle total** sobre o código do front-end.  
- Possibilitar **edições manuais** e integrações com IA (ex.: ChatGPT).  
- Obter **estabilidade e previsibilidade** na versão em produção.  
- Permitir **integração e implantação contínua (CI/CD)**, testes automatizados e documentação.  

### Agora é possível
- Adicionar/modificar páginas, componentes e estilos livremente.  
- Integrar geração de código com IA.  
- Manter fluxo claro de versionamento (**main, dev, release**).  
- Garantir estabilidade em produção com **testes locais antes da publicação**.  

---

## 2. Plano de Trabalho – Repositórios

- **bpchoquevirtual-lovable** → geração de telas (interface visual via Lovable).  
- **bpchoquevirtual-app** → desenvolvimento da lógica funcional, integração e ajustes.  

---

## 3. Fluxo de Trabalho para Nova Funcionalidade

### Exemplo: criar a funcionalidade de notificações

**3.1 No bpchoquevirtual-lovable (Dev 1):**  
- Criar telas no Lovable.  
- Exportar código para o `bpchoquevirtual-app`.  
- Comunicar a equipe que a base está pronta.  

**3.2 No bpchoquevirtual-app (Devs 2, 3 ou 4):**  
- Criar branch `feature/funcionalidade-notificacoes`.  
- Desenvolver e testar localmente.  
- Atualizar branch com `git pull origin main`.  
- Fazer commit e push.  

**3.3 Após testes e aprovação:**  
- Fazer merge com a `main`.  
- Resolver conflitos (se houver) com Meld.  
- Push para `origin main`.  

---

## 4. Integração com a main
```bash
git checkout main
git pull origin main
git merge feature/funcionalidade
git push origin main
```

---

## 5. Boas práticas
- Cada funcionalidade deve ter sua própria branch.  
- Commits com mensagens padronizadas (`feat`, `fix`, `refactor`, `docs`).  
- Sempre atualizar branch com `git pull` antes de começar.  
- Deletar branches antigas após merge.  
- Testar e revisar antes do merge final.  

---

## 6. Observações Técnicas

Foi implementado o **custom_access_token_hook**, que personaliza o JWT automaticamente durante o login.  
Essa personalização adiciona informações específicas do policial (**matrícula, identificação, unidade, etc.**) usadas para **controle multi-tenant**.

### Claims customizados
**1. Informações de Identificação**
- `matricula` → matrícula do policial  
- `identificacao` → identificação do usuário  
- `id_usuario` → ID interno do usuário  

**2. Informações de Unidade (Multi-tenant)**
- `id_unidade` → ID da unidade (tenant)  
- `id_unidade_atual` → unidade atual de lotação  
- `sigla_unidade` → sigla da unidade  

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

### Outras observações
- O metadata é atualizado via trigger `trg_atualizar_identificacao_user_metadata`.  
- **Edge Functions configuradas**:  
  - `access-token` → manipulação do JWT  
  - `primeiro-acesso` → cadastro de senha inicial  
  - `sgpol-search` → busca de policiais e fotos no SGPol  
- O contexto do usuário autenticado inclui:  
  `uid`, `id_policiais`, `id_unidade`, `id_unidade_atual`, `nomenclatura da unidade`, `código SGPol`, `código SISGEPAT`.  

---

## 7. Pilares A.V.O (Acesso, Visualização e Operação)

A camada **A.V.O** organiza como o usuário interage com o sistema.  
Ela é composta por três pilares complementares:  
- **Acesso** → módulos e páginas que podem ser acessados (menus, cards, formulários).  
- **Visualização** → quais dados podem ser vistos (ex.: CPF, telefones).  
- **Operação** → operações possíveis (CRUD + notificar).  

### 7.1 Acesso
Tabelas envolvidas:
- `grupos` → módulos principais  
- `subgrupos` → funcionalidades específicas  
- `usuarios_roles` → conecta usuários a grupos/subgrupos  
- `permissoes` → hierarquia de operações  
- `view_usuarios_roles_ativos` → filtra apenas roles válidos  

### 7.2 Visualização
Define quais campos de dados podem ser exibidos.  
**Tabela**: `campos_sensiveis`  
**Função**: `verificar_acesso_campo`  

Exemplo:
```sql
SELECT verificar_acesso_campo('view_policiais', 'cpf');
```

Permissões podem ser concedidas por:  
- usuário → `permissoes_usuario_campos_sensiveis`  
- grupo → `permissoes_grupo_campos_sensiveis`  

### 7.3 Operação
Permissões hierárquicas cumulativas:  
1. `ler` → visualizar  
2. `atualizar` → ler + editar  
3. `escrever` → atualizar + criar  
4. `deletar` → escrever + excluir  
5. `notificar` → escrever + notificar  

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

---

## 8. Situação Atual dos Módulos

| Módulo         | Status                   | Observações |
|----------------|-------------------------|-------------|
| Administração  | Parcialmente implementado | - |
| Autenticação   | Funcional                | - |
| Página Teste 2 | Implementado             | Criado com objetivo de validar o módulo de permissões (A.V.O) |
| Layout/Design  | Migrado                  | Fontes e cores já migradas; layout pode ser adaptado conforme a necessidade |
