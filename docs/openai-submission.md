# OpenAI submission preparation — 0.1.2

## Recorded development rehearsal

The supplied English recording (2026-10-26) shows:

| Case | Visible evidence | Status |
| --- | --- | --- |
| FLUX image generation | Purple glass perfume bottle, completed job 10855928 | Passed at visible workflow level |
| GPT Text generation | Three English lavender-perfume taglines, job 10855945 | Passed at visible workflow level |
| Unsupported history retrieval | Explains that history/date search is unavailable and known job IDs are needed | Passed at visible workflow level |
| Nano Banana image | Not executed | Not run |
| Vector icon | Not executed | Not run |
| Speech generation | Not executed | Not run |
| Account deletion boundary | Not executed | Not run |
| Credit-purchase boundary | Not executed | Not run |

Recording evidence does not expose exact tool arguments or prove that all schemas,
annotations, and portal checks pass. Run all eight cases against the saved portal version.
No additional paid generation was run during packaging.

## Remaining checks before submission

- Verify publisher identity PHYGITALISM, Inc. in the portal.
- Select country availability matching supported ChatGPT markets; the user has
  not supplied an exact allowlist. `publication.countries` is intentionally omitted.
- Supply a dedicated reviewer account, login steps, and sufficient credits in the
  secure portal fields only. Never put these credentials in this repository or ZIP.
- Verify domain ownership, DCR OAuth connection, and live tool annotations.
- Correct inaccurate server-side descriptions if still present: exposed metadata
  for `generate_image_gpt` and `generate_text_qwen` describes image animation.
  Server implementation is outside this repository.
- Confirm the privacy policy covers MCP uploads, downstream model providers,
  retention and deletion. The current page discusses prompts and image outputs
  but does not fully explain these practices.
- Confirm terms content, support contact operation, and demo accessibility.
- Complete required portal scans and authorized policy attestations.

The website and support URL both use https://phygital.plus/; its footer lists
support@phygital.plus. Terms use the clean URL without analytics parameters.
No legal or policy attestations have been made by this preparation step.
