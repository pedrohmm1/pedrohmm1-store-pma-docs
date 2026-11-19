# store-pma · documentação individual (Pedro Machado)

Documentação MkDocs da disciplina **Platforms, Microservices, DevOps and APIs** (Insper, 2026.2):
projeto [store-pma](https://github.com/EP-PC/store-pma) e atividades individuais.

**Site:** https://pedrohmm1.github.io/pedrohmm1-store-pma-docs/

Rodar localmente:

```bash
python -m venv .venv
.venv/Scripts/pip install -r requirements.txt     # Linux/macOS: .venv/bin/pip
.venv/Scripts/mkdocs serve                        # http://127.0.0.1:8000
```

Publicação: GitHub Actions (`.github/workflows/main.yaml`) roda `mkdocs gh-deploy` a cada push na `main`.
