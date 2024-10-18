fbzdf# Test Repository for GitHub Secrets Scanner

This is a test repository to test the GitHub Secrets Scanner Action.
this is a testm
adfB
name: 'Secrets Scanner Test Workflow'
on: [push, pull_request]
jobs:
  test_secrets_scanner:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3  # Correctly refer to an official GitHub Action to checkout the code

      - name: Set up Python
        uses: actions/setup-python@v2  # Correct reference to set up Python environment
        with:
          python-version: '3.x'

      - name: Install TruffleHog
        run: |
          python --version  # Check Python version
          pip install trufflehog  # Install TruffleHog using pip

      - name: Install Gitleaks
        run: |
          echo "Installing Gitleaks..."
          curl -sSfL https://github.com/zricethezav/gitleaks/releases/download/v8.5.0/gitleaks_8.5.0_linux_x64.tar.gz | tar -xz -C /usr/local/bin
          chmod +x /usr/local/bin/gitleaks
          gitleaks version  # Verify Gitleaks installation

      - name: Debug Directory
        run: |
          echo "Current directory content:"
          ls -la

      - name: Run TruffleHog Secrets Scanner
        id: trufflehog_scan
        run: |
          echo "Running TruffleHog..."
          trufflehog . > trufflehog-results.txt || echo "No secrets detected by TruffleHog." > trufflehog-results.txt
          if [ ! -s trufflehog-results.txt ]; then
            echo "No secrets detected by TruffleHog." > trufflehog-results.txt
          fi

      - name: Run Gitleaks Secrets Scanner
        id: gitleaks_scan
        run: |
          echo "Running Gitleaks..."
          gitleaks detect -v --source . --report-format json --report-path gitleaks-results.json || echo '[]' > gitleaks-results.json
          if [ ! -s gitleaks-results.json ]; then
            echo '[]' > gitleaks-results.json  # Create empty JSON if no secrets detected
          fi

      - name: Upload TruffleHog Results
        if: always()
        uses: actions/upload-artifact@v3  # Correctly refer to upload-artifact action
        with:
          name: trufflehog-results
          path: trufflehog-results.txt

      - name: Upload Gitleaks Results
        if: always()
        uses: actions/upload-artifact@v3  # Correct reference for uploading artifacts
        with:
          name: gitleaks-results
          path: gitleaks-results.json

      - name: Send Slack Notification
        if: always()  # Always run, regardless of success or failure
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        run: |
          # Check if any secrets were detected by either scanner
          if grep -q -v 'No secrets detected by TruffleHog' trufflehog-results.txt || ([ -s gitleaks-results.json ] && [ "$(cat gitleaks-results.json)" != "[]" ]); then
            MESSAGE="🚨 Secrets were detected in the repository! Please review the scan results."
          else
            MESSAGE="✅ No secrets detected by either TruffleHog or Gitleaks. All clear!"
          fi

          # Check if SLACK_WEBHOOK_URL is set
          if [ -z "$SLACK_WEBHOOK_URL" ]; then
            echo "Error: SLACK_WEBHOOK_URL is not set"
            exit 1
          fi

          # Send Slack notification using the curl command
          curl -X POST -H 'Content-type: application/json' --data "{\"text\":\"${MESSAGE}\"}" "$SLACK_WEBHOOK_URL"

KLL;
zd fc
s era redrd z
rt d
v z
