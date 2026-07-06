# Generate Docs

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve --config-file mkdocs.yml
mkdocs build -f mkdocs.yml
```

This build deletes all important **docs/CNAME**.
This file enables redirection from
<https://prasadtalasila.github.io> to
<https://prasad.talasila.in>.

```bash
cp CNAME docs/CNAME
git add
git commit -m "..."
git push origin
```
