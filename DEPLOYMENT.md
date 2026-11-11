# Publish Jersey Pattern Extractor online

This project is configured for Vercel. Your users will access a public website URL; they will not run anything locally.

## Steps

1. Put all project files in a GitHub repository.
2. Open https://vercel.com and sign in.
3. Select **Add New → Project** and import the repository.
4. Keep the detected framework as **Next.js**.
5. Add these Environment Variables in the Vercel project settings:

| Name | Value |
|---|---|
| `ANTHROPIC_BASE_URL` | `https://claude.dramakuy.id/anthropic` |
| `ANTHROPIC_API_KEY` | Your rotated private key |
| `ANTHROPIC_VERSION` | `2023-06-01` |
| `ANTHROPIC_MODEL` | Exact model alias supported by your gateway |

6. Select **Deploy**.
7. Wait for the deployment to finish. Vercel will provide a URL such as `https://your-project.vercel.app`.

## Security

- Do not commit `.env.local`.
- Do not use `NEXT_PUBLIC_` for the API key.
- Rotate the key previously shared in chat.
- Add authentication/rate limiting before opening the tool to the public.

## Upload size

The current Vercel serverless MVP accepts images up to 4 MB. For larger jersey mockups, implement direct upload to Vercel Blob or Cloudflare R2.
