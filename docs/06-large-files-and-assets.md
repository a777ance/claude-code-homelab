# 06 — Large Files, Assets, and Copyright

Lessons learned working with large external datasets, reference material, and third-party assets inside Claude Code projects.

---

## Remote sessions have restricted network access

Claude Code cloud sessions run in isolated containers. Outbound network access may be blocked or filtered depending on the environment policy. This means:

- `WebFetch` (Claude's built-in fetch tool) will return 403 or 404 for many public sources — even legitimate ones like Project Gutenberg or direct CDN URLs
- Claude **cannot reliably download large files** on your behalf in a remote session
- The fix: push a **download script** to the repo and run it locally

**Pattern:**
```
reference/
  some-dataset/
    README.md       ← documents source, license, format
    download.py     ← fetches and formats the data
    data.txt        ← gitignored until you run the script, then committed
```

This is more reproducible than having Claude paste in raw content anyway.

---

## Never pass large files through Claude's context

Claude has a context window. Trying to load a 4MB text file into it via WebFetch will result in truncation or summarization — you will not get the full content.

Rule of thumb:
- **Under ~50KB**: Claude can read it directly
- **50KB–500KB**: possible but degrades context for other work
- **Over 500KB**: use a script; do not attempt to load inline

For very large assets (datasets, corpora, media), use a download script and commit the result. Let the file live in the repo — not in Claude's context.

---

## GitHub file size limits

| Threshold | Behavior |
|-----------|----------|
| Under 50MB | Fully supported |
| 50MB–100MB | Allowed but GitHub warns |
| Over 100MB | Push rejected |
| Over 100MB (needed) | Use [Git LFS](https://git-lfs.com/) |

For binary assets (images, audio, video, trained models), Git LFS is the right tool regardless of size.

---

## Copyright: always check before committing

Committing copyrighted content to a repo — even a private one — creates legal exposure.

### Text

| Safe to commit | Not safe |
|---------------|----------|
| Public domain works (US: published before 1928, or government works) | Modern translations (NIV, ESV, NASB, NKJV, NLT, The Message) |
| Creative Commons licensed text (check the specific CC variant) | Academic papers without open access license |
| KJV Bible (1611), ASV (1901), WEB, YLT | Most post-1928 novels, articles, lyrics |

### Images and artwork

| Safe to commit | Not safe |
|---------------|----------|
| Works by artists dead 70+ years (EU rule) or published before 1928 (US rule) | Stock photos without license |
| Wikimedia Commons files marked PD or CC0 | AI-generated images with unclear terms |
| US government photographs | Photographs of public domain artworks (Bridgeman ruling applies in US; check jurisdiction) |

### APIs and data

- Fetching is not the same as committing — check the API's ToS for redistribution rights
- Some APIs allow personal use but prohibit storing their data in a repo
- When in doubt: store a download script, not the data

---

## Immutable reference material

Some content should be committed once and never modified. Examples: canonical texts, legal documents, datasets used as ground truth.

Conventions to enforce this:
- Add a `README.md` in the directory stating the immutability rule explicitly
- Use a commit message like `reference: commit [source] — do not modify`
- Consider adding the file(s) to `.gitattributes` with a custom note, or using branch protection rules
- In creative/research projects: name the principle explicitly (e.g., "Sola Scriptura") so all contributors understand the rule and its reason

---

## Layer separation

In projects that mix reference material, theory, and output:

```
reference/     ← external source material, immutable
principles/    ← theory, frameworks, organizing ideas (mutable, living)
manuscript/    ← the actual output (the work being produced)
```

Keeping these layers separate prevents theoretical scaffolding from contaminating the work, and prevents the work from being constrained prematurely by theory. Any layer can change without touching the others.

---

## Checklist before committing a large or third-party asset

- [ ] Is the license confirmed? (public domain, CC0, or explicit permission)
- [ ] Is the file under 50MB? (use Git LFS if not)
- [ ] Is there a README documenting the source and any immutability rules?
- [ ] If it required a download script: is the script committed too?
- [ ] Does `.gitignore` handle any intermediate or generated files?
