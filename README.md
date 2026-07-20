# plan-twa — app Android (TWA) do PLAN

Empacota o PWA `https://planescala.com.br` como app Android (Trusted Web Activity) via
[Bubblewrap](https://github.com/GoogleChromeLabs/bubblewrap). O build roda **no GitHub Actions**
(runner Ubuntu) — não depende de disco/toolchain no Mac.

- `twa-manifest.json` — **fonte da verdade** da configuração do app (package, cores, ícones, atalhos).
- `.github/workflows/android-build.yml` — gera o projeto e builda o `.aab` assinado.
- O projeto Android (`app/`, `build.gradle`, `gradlew`…) é **regerado** no CI por `bubblewrap update`; por isso está no `.gitignore`.

## Dados fixos
- Package / applicationId: `br.com.planescala.app` (imutável)
- Nome: **PLAN** · display standalone · portrait · cor `#101828`
- Keystore SHA-256 (upload key): `67:0D:4F:BA:B1:B4:51:4D:A0:CB:8A:25:2C:31:5C:41:02:DF:77:16:43:59:6C:21:66:63:9E:49:3F:79:1A:36`

## Segredos do repositório (GitHub → Settings → Secrets and variables → Actions)

| Segredo | Valor |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | base64 do arquivo `android.keystore` (ver abaixo) |
| `ANDROID_KEYSTORE_PASSWORD` | a senha da keystore (guardada no cofre) |
| `ANDROID_KEY_PASSWORD` | mesma senha (store e key são iguais) |

Gerar o base64 da keystore (no Mac, onde ela está):
```bash
base64 -i ~/plan-twa/android.keystore | pbcopy   # cola em ANDROID_KEYSTORE_BASE64
```

## Rodar o build
**Actions → "Android TWA — build AAB" → Run workflow** (ou `git tag android-v1 && git push --tags`).
O `.aab` sai como *artifact* do run (aba Actions → run → Artifacts → `android-aab-apk`).

## Publicar na Play Store
1. Criar o app no Play Console (US$ 25 único). Ativar **Play App Signing** (recomendado).
2. Subir o `.aab`. Pegar o **SHA-256 da chave de assinatura do app** em
   *Play Console → Integridade do app → Assinatura do app*.
3. Colocar esse SHA-256 (e o de upload acima) na env `ANDROID_ASSETLINKS_SHA256` do `.env` de produção
   do Django e redeployar → o app abre em tela cheia (sem a barra do Chrome).
4. Conta pessoal pós-2023: teste fechado com ≥12 testadores por 14 dias antes de produção.
