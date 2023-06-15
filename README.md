<h1 align="center">PureDNS Action</h1>

Example Usage
-----

**GitHub Action running `puredns` to resolve subdomains**

```yaml
      - name: Run PureDNS to Resolve
        uses: KickAss101/puredns-action@main
        with:
          list: input/subdomains.txt
          resolvers: resolvers/resolvers.txt
          trusted_resolvers: resolvers/resolvers-trusted.txt
```

**GitHub Action running `puredns` to bruteforce subdomains**

```yaml
      - name: Run PureDNS to Brute
        uses: KickAss101/puredns-action@main
        with:
          domain: hackerone.com
          resolvers: resolvers/resolvers.txt
          trusted_resolvers: resolvers/resolvers-trusted.txt
          dns_list: wordlists/dns-words.txt
          bruteforce: 'true'
          rate_limit: 750
```

**Example workflow to resolve and bruteforce simultaneously**: `.github/workflows/puredns.yml`



```yaml
name: Recon Action

on:
  push:
    branches:
      - main

permissions: write-all

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Go environment
        uses: actions/setup-go@v4.0.1

      - uses: actions/setup-go@v4
        with:
          check-latest: true
          cache-dependency-path: subdir/go.sum
      - run: go version

      # Upload Artifacts
      - name: Upload artifacts
        uses: actions/upload-artifact@v2
        with:
          name: artifacts
          path: artifacts

  resolve-subdomains:
    needs: build
    runs-on: ubuntu-latest
  
    steps:
      # Download Artifacts
      - name: Download artifact
        uses: actions/download-artifact@v2
        with:
          name: artifacts
          path: artifacts

      - name: Run PureDNS to Resolve
        uses: KickAss101/puredns-action@main
        with:
          list: artifacts/input/subdomains.txt
          resolvers: artifacts/resolvers/resolvers.txt
          trusted_resolvers: artifacts/resolvers/resolvers-trusted.txt

      - name: Upload Resolve Artifacts
        uses: actions/upload-artifact@v2
        with:
          name: artifacts
          path: subs.resolve

      - name: Upload Wildcards Artifacts
        uses: actions/upload-artifact@v2
        with:
          name: artifacts
          path: subs.wildcards

  brute-force-subdomains:
    needs: build
    runs-on: ubuntu-latest

    steps:
      # Download Artifacts
      - name: Download artifact
        uses: actions/download-artifact@v2
        with:
          name: artifacts
          path: artifacts

      - name: Run PureDNS to Brute
        uses: KickAss101/puredns-action@main
        with:
          resolvers: artifacts/resolvers/resolvers.txt
          trusted_resolvers: artifacts/resolvers/resolvers-trusted.txt
          domain: hilton.com
          dns_list: artifacts/wordlists/words.txt
          bruteforce: 'true'
          rate_limit: 1500

      - name: Upload Bruteforce Artifacts
        uses: actions/upload-artifact@v2
        with:
          name: artifacts
          path: subs.brute

  commit-and-push:
    needs: [resolve-subdomains, brute-force-subdomains]
    if: ${{ needs.resolve-subdomains.result == 'success' || needs.brute-force-subdomains.result == 'success' }}
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      # Download Artifacts
      - name: Download artifact
        uses: actions/download-artifact@v2
        with:
          name: artifacts
          path: artifacts

      - name: Create local changes
        run: |
          git add artifacts/subs.resolve
          git add artifacts/subs.wildcards
          git add artifacts/subs.brute

      - name: Commit results to GitHub
        run: |
          git config --local user.email "jaswanthsunkara@protonmail.com"
          git config --global user.name "KickAss101"
          git commit -am "Recon report" --allow-empty

      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.PAT_TOKEN }}
          branch: ${{ github.ref }}

```
![image](https://github.com/KickAss101/puredns-action/assets/46389158/c34597d4-e9ba-45f1-bc36-87261c42441e)

Available Inputs
------

| Key                 | Description                     | Required |
|---------------------|---------------------------------|----------|
| `list`              | List of hosts to resolve        | false    |
| `domain`            | Domain to run bruteforce        | false    |
| `dns_list`          | DNS words to run bruteforce     | false    |
| `bruteforce`        | Bruteforce sub command          | false    |
| `rate_limit`        | Queries/sec for public resolvers| false    |
| `resolvers`         | List of public resolvers        | true     |
| `trusted_resolvers` | List of trusted resolvers       | true     |





