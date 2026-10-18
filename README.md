# Gerenciando Políticas e Acessos no Azure

Este repositório documenta os principais conceitos estudados no desafio **Gerenciando Políticas em Acessos Azure**, da **DIO (Digital Innovation One)**.

O objetivo é compreender como organizar recursos e aplicar controles de **governança, identidade, permissões e conformidade** no Microsoft Azure, reduzindo riscos operacionais e de segurança.

> Material de estudo: os exemplos são ilustrativos e não representam, necessariamente, recursos provisionados durante o laboratório.

## Objetivos de aprendizagem

- Entender a organização hierárquica de recursos na Azure.
- Distinguir gerenciamento de identidades, permissões e políticas.
- Conhecer Azure RBAC, Azure Policy e bloqueios de recursos.
- Aplicar o princípio do menor privilégio.
- Compreender o papel das tags, grupos de gerenciamento e logs.
- Identificar boas práticas de governança e auditoria.

## 1. Organização dos recursos Azure

A Azure possui uma estrutura hierárquica:

```text
Microsoft Entra Tenant
└── Management Groups
    └── Subscriptions
        └── Resource Groups
            ├── Virtual Machines
            ├── Storage Accounts
            └── Outros recursos
```

Os **Management Groups** permitem aplicar governança sobre várias assinaturas. Os **Resource Groups** organizam recursos relacionados, facilitando gerenciamento, permissões e acompanhamento.

## 2. Microsoft Entra ID

O **Microsoft Entra ID** (antigo Azure Active Directory) fornece recursos de identidade, autenticação e controle de acesso a aplicativos. Usuários, grupos e identidades de aplicações podem ser usados para solicitar acesso a serviços da Azure.

**Autenticação** verifica quem está se conectando. **Autorização** determina quais operações essa identidade pode realizar.

## 3. Controle de acesso com Azure RBAC

O **Azure Role-Based Access Control (RBAC)** utiliza atribuições de funções para conceder permissões sobre recursos do Azure.

Cada atribuição combina:
- **Principal de segurança:** usuário, grupo, entidade de serviço ou identidade gerenciada.
- **Função (role):** conjunto de operações autorizadas.
- **Escopo:** grupo de gerenciamento, assinatura, grupo de recursos ou recurso.

### Funções comuns

| Função | Finalidade |
| --- | --- |
| Reader | Visualizar recursos, sem alterá-los |
| Contributor | Gerenciar recursos, mas sem conceder acesso via atribuição de funções |
| Owner | Gerenciar recursos e permissões de acesso |
| User Access Administrator | Gerenciar o acesso de usuários aos recursos |

As funções e permissões devem ser verificadas conforme a necessidade. O ideal é conceder apenas as permissões necessárias no menor escopo possível.

## 4. Azure Policy

O **Azure Policy** auxilia na aplicação de regras de governança e na avaliação da conformidade de recursos.

Exemplos:
- permitir criação de recursos apenas em determinadas regiões;
- exigir tags como `ambiente` e `centro-de-custo`;
- auditar recursos fora dos padrões;
- negar configurações que violem regras estabelecidas.

Uma definição de política contém regras e um **efeito**, que pode incluir `audit`, `deny`, `modify` ou `deployIfNotExists`, conforme a situação e os requisitos.

### Azure RBAC x Azure Policy

| Recurso | Pergunta principal |
| --- | --- |
| Azure RBAC | **Quem** pode realizar determinada operação? |
| Azure Policy | **Quais configurações** de recursos são permitidas ou estão em conformidade? |

Um usuário com permissão para criar uma VM ainda pode ter a solicitação bloqueada por uma política que proíba a região escolhida.

## 5. Policy Initiatives

Uma **initiative** agrupa diversas definições de políticas para aplicar e acompanhar um conjunto de requisitos.

Por exemplo, uma iniciativa para um ambiente corporativo pode verificar tags obrigatórias, regiões permitidas e configurações de segurança.

Isso simplifica o acompanhamento da conformidade em várias assinaturas ou grupos de recursos.

## 6. Bloqueios de recursos (Resource Locks)

Os bloqueios ajudam a prevenir alterações ou exclusões acidentais:

- **CanNotDelete:** permite alterações autorizadas, mas bloqueia exclusão do recurso.
- **ReadOnly:** bloqueia operações de modificação no plano de controle, inclusive algumas operações que parecem de leitura no uso cotidiano.

Bloqueios podem ser aplicados em diferentes escopos e herdados pelos recursos abaixo deles. Não substituem backups, controles de identidade ou proteção no plano de dados.

## 7. Tags e organização

As **tags** identificam recursos para fins de administração e análise.

Exemplo:

```text
ambiente = desenvolvimento
projeto = desafio-dio
responsavel = equipe-cloud
centro-de-custo = treinamento
```

Elas ajudam a filtrar recursos e analisar gastos. Entretanto, nem todos os tipos de recursos ou itens cobrados oferecem suporte à mesma propagação de tags.

## 8. Acesso Condicional e MFA

O **Microsoft Entra Conditional Access** permite aplicar decisões de acesso com base em condições como identidade, risco, dispositivo e localização, de acordo com as capacidades licenciadas.

A **autenticação multifator (MFA)** reduz riscos associados a credenciais comprometidas. O uso de MFA é especialmente importante para contas administrativas.

## 9. Privileged Identity Management (PIM)

O **Microsoft Entra Privileged Identity Management (PIM)** possibilita administrar acessos privilegiados com recursos como ativação temporária, exigência de justificativa e aprovação em cenários configurados.

O objetivo é reduzir a permanência de permissões elevadas. A disponibilidade depende de licenciamento e do tipo de papel gerenciado.

## 10. Monitoramento e auditoria

Os registros de atividades e de autenticação apoiam investigações e verificações de conformidade:

- **Azure Activity Log:** registra eventos do plano de controle da assinatura.
- **Microsoft Entra Sign-in Logs:** registra eventos de entrada e autenticação.
- **Azure Monitor / Log Analytics:** oferece análise e alertas quando as fontes estão configuradas.
- **Azure Policy Compliance:** apresenta avaliação de conformidade dos recursos às políticas atribuídas.

Logs não substituem controles preventivos e devem ter retenção e acesso planejados.

## Exemplo prático de governança

Imagine uma equipe com ambientes de desenvolvimento e produção:

```text
Management Group
└── Assinatura
    ├── RG-Desenvolvimento
    │   ├── Devs: permissões limitadas
    │   └── Tags: ambiente=dev
    └── RG-Producao
        ├── Operadores: Reader
        ├── Admins: permissões controladas
        └── Tags: ambiente=prod

Azure Policy → Regiões e tags permitidas
Azure RBAC   → Papéis e escopos de acesso
Resource Lock → Proteção contra exclusões
Azure Monitor → Auditoria e alertas
```

Assim, diferentes ferramentas atuam juntas: **RBAC limita quem pode agir**, **Policy avalia ou restringe configurações** e **Locks ajudam a evitar mudanças acidentais**.

## Boas práticas

1. Usar grupos de usuários para administrar permissões sempre que adequado.
2. Aplicar o princípio do menor privilégio.
3. Evitar permissões permanentes de Owner sem necessidade.
4. Exigir MFA para contas privilegiadas.
5. Definir padrões de tags e nomenclatura.
6. Validar políticas inicialmente em modo de auditoria quando houver risco de impacto operacional.
7. Proteger recursos críticos contra exclusão acidental.
8. Revisar periodicamente as atribuições RBAC e exceções de Policy.
9. Monitorar logs relevantes e respostas a alertas.
10. Documentar escopos, responsáveis e justificativas para permissões elevadas.

## Conclusão

Gerenciar políticas e acessos no Azure não consiste apenas em liberar ou negar permissões. Uma estratégia consistente combina **identidade, autorização, padrões de configuração, auditoria e proteção contra alterações indevidas**.

O principal aprendizado é que **governança e segurança precisam ser planejadas desde a criação do ambiente**, com controles proporcionais às necessidades de cada aplicação.

## Referências

- [Azure RBAC](https://learn.microsoft.com/pt-br/azure/role-based-access-control/overview)
- [Azure Policy](https://learn.microsoft.com/pt-br/azure/governance/policy/overview)
- [Management Groups](https://learn.microsoft.com/pt-br/azure/governance/management-groups/overview)
- [Bloqueios de recursos](https://learn.microsoft.com/pt-br/azure/azure-resource-manager/management/lock-resources)
- [Microsoft Entra ID](https://learn.microsoft.com/pt-br/entra/fundamentals/whatis)
- [Acesso Condicional](https://learn.microsoft.com/pt-br/entra/identity/conditional-access/overview)
- [Microsoft Entra PIM](https://learn.microsoft.com/pt-br/entra/id-governance/privileged-identity-management/pim-configure)
- [Azure Monitor](https://learn.microsoft.com/pt-br/azure/azure-monitor/overview)

---

**Desafio de Projeto — DIO**
