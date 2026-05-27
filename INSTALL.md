# 🚀 Guia de Onboarding e Instalação

Este documento detalha o funcionamento dos utilitários de automação do repositório de skills.

## Como funciona a sincronização?

O script `install.py` faz a ponte entre este repositório central de desenvolvimento de skills e a pasta de configuração local do seu agente de IA:
- No Windows: `C:\Users\<user>\.gemini\config\skills`
- No macOS/Linux: `~/.config/skills`

---

## Comandos do `install.py`

### 1. Instalar TODAS as skills do repositório
```bash
python install.py --all
```
Este comando copia as pastas de cada skill, valida suas regras internas, registra-as na CLI local do Claude Code (`.claude/skills`) e reconstrói o `registry.json` do orquestrador.

### 2. Instalar uma skill específica
```bash
python install.py --skill stack-conventions
```

### 3. Validar a integridade de todas as skills locais
```bash
python install.py --validate
```
Executa 10 testes rigorosos em cada arquivo `SKILL.md` (frontmatter YAML parseável, nome em minúsculas, tamanho máximo, ausência de tokens ou arquivos confidenciais).

---

## Geração de Pacotes para Interface Web (`package.py`)

Para exportar as skills para uso no painel web (Claude.ai):
```bash
# Empacota todas as skills
python package.py --all

# Empacota apenas uma
python package.py --skill zero-downtime-migrations
```
Isso criará arquivos ZIP higienizados (com barras normalizadas e sem arquivos de sistema) na pasta `/dist`.
