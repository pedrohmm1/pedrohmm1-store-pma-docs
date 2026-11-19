# Esta documentação

Feita com [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), a partir do
[template da disciplina](https://hsandmann.github.io/documentation.template/), e publicada no
GitHub Pages pelo workflow `.github/workflows/main.yaml` (`mkdocs gh-deploy` a cada push na `main`).
A estrutura das páginas segue a da documentação do Pedro Garcia, para as três documentações do
grupo terem o mesmo formato.

Os vídeos em `docs/video/` são do grupo (gravados pelo Pedro Garcia).

## Rodar localmente

```bash
python -m venv .venv
.venv/Scripts/pip install -r requirements.txt     # Linux/macOS: .venv/bin/pip
.venv/Scripts/mkdocs serve                        # http://127.0.0.1:8000
```

## Publicar

1. Push na `main`: o GitHub Actions builda o site e publica na branch `gh-pages`.
2. Uma vez só: em **Settings → Pages** do repositório, escolher *Deploy from a branch* →
   `gh-pages` / `root`.
3. Em **Settings → Actions → General → Workflow permissions**: *Read and write permissions*.
