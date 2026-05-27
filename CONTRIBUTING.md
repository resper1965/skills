# 🛠️ Como Contribuir para o Repositório de Skills

Obrigado pelo seu interesse em expandir o **Enterprise SaaS Agentic Ecosystem**. 
Nós utilizamos um processo rigoroso de Quality Gates para garantir que todas as 258+ skills mantenham o nível de excelência e não quebrem a execução dos agentes locais.

## 📋 Regra de Ouro: YAML Frontmatter
Toda skill **DEVE** possuir um arquivo `SKILL.md` na raiz do seu diretório. 
Este arquivo **DEVE** começar com um YAML Frontmatter válido contendo o `name` e a `description` da skill.

Exemplo de formatação obrigatória no início do `SKILL.md`:
```yaml
---
name: minha-nova-skill
description: Breve descrição em uma linha sobre o objetivo da skill.
---

# Título Principal da Skill
Corpo da documentação usando formatação Markdown...
```

**Por que isso é necessário?**
1. O validador CI (`validate.py`) verifica a integridade desses metadados.
2. O catálogo dinâmico (`CATALOG.md`) utiliza essas informações para exibir os links de forma categorizada.
3. O orquestrador de IA utiliza essas strings para realizar o *semantic routing* correto da sua skill.

## 🔒 Auditoria de Segurança (Secret Leakage)
Não comite tokens, senhas, chaves de API, ou arquivos confidenciais. 
O script de validação rodará antes de todo commit (via pre-commit hook) e bloqueará alterações caso arquivos não autorizados ou padrões de secrets sejam detectados.

## ⚙️ Criando sua Skill Passo a Passo

1. Crie uma pasta dentro do diretório `/skills/` usando letras minúsculas e hífens:
   ```bash
   mkdir skills/nova-arquitetura-db
   ```
2. Crie o arquivo `SKILL.md` lá dentro.
3. Escreva as regras para a IA de forma objetiva (pense em como um modelo de linguagem interpreta instruções).
4. Rode o gerador de catálogo para atualizar a documentação central:
   ```bash
   python generate_catalog.py
   ```
5. Faça o commit da sua nova skill!

## 🧪 Validação Local
Se quiser testar a validação antes de comitar:
```bash
python validate.py --repo-check
```
Se a saída for `[OK] All skills are healthy`, você está pronto para subir seu código!
