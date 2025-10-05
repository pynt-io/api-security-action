<div align="center">
  <img src="https://cdn.prod.website-files.com/6357c772bd0a59790d6cfdf5/678a156ed7dfe8140e06084e_Pynt-logo.svg" alt="Pynt Logo" width="200"/>

  # Pynt API Security Action

  [![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Pynt%20API%20Security-blue.svg)](https://github.com/marketplace/actions/pynt-api-security-action)
</div>

---

Run [Pynt](https://pynt.io) API security testing in your GitHub workflows using the Pynt Linux binary. This action supports two modes:
- **Newman mode**: Security testing with Postman collections
- **Command mode**: Security testing with any test framework (Selenium, RestAssured, Jest, Pytest, Go, JMeter, etc.)

> **Note**: This action uses the Pynt Linux binary and is designed for Linux runners (ubuntu-latest).

## Features

- ✅ Automated API security testing in CI/CD
- ✅ Support for Postman/Newman collections
- ✅ Framework-agnostic command mode (works with any testing framework)
- ✅ Comprehensive vulnerability scanning
- ✅ Configurable severity levels and reporting
- ✅ mTLS support
- ✅ SSL/TLS certificate handling

## Prerequisites

- A Pynt account and Pynt ID ([Get started](https://pynt.io))
- Get your Pynt ID by running: `pynt pynt-id`
- Store your Pynt ID as a GitHub secret named `PYNT_ID`

## Usage

### Newman Mode

Test your Postman collections with Pynt security scanning:

```yaml
- name: Run Pynt with Newman
  uses: pynt-io/api-security-action@v1
  with:
    mode: newman
    pynt-id: ${{ secrets.PYNT_ID }}
    collection: ./postman/api-collection.json
    environment: ./postman/env.json
    test-name: "API Security Scan"
```

> **Note**: The `pynt-id` input is required. Pass it from your GitHub secrets.

### Command Mode

Run security testing alongside your existing functional tests:

```yaml
- name: Run Pynt with Pytest
  uses: pynt-io/api-security-action@v1
  with:
    mode: command
    pynt-id: ${{ secrets.PYNT_ID }}
    command: "pytest tests/api_tests.py"
    application-name: "API Security Scan"
```

## Inputs

### Common Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `mode` | Pynt mode: `newman` or `command` | **Yes** | - |
| `pynt-id` | Pynt ID for authentication | **Yes** | - |
| `application-name` | Associate scan with a specific application (creates application if it does not exist) | No | - |
| `severity-level` | Return error code on findings: `all`, `medium`, `high`, `critical`, `none` (default: none means no error on findings) | No | `none` |
| `tags` | Comma-separated tags for the scan | No | - |
| `report-path` | Path to save the generated report | No | - |
| `host-ca` | Path to CA file in PEM format for SSL verification | No | - |
| `verbose` | Get more detailed run information | No | `false` |

### Newman Mode Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `collection` | Path to Postman collection file | **Yes** |
| `environment` | Path to Postman environment file(s), comma-separated for multiple | No |
| `newman-env-var` | Environment variables (e.g., `VAR1=value1,VAR2=value2`) | No |
| `tls-client-cert` | Path to client certificate for mTLS | No |
| `tls-client-key` | Path to client private key for mTLS | No |

### Command Mode Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `command` | Command that runs your functional tests | **Yes** |
| `allow-errors` | Continue execution if command fails | No |
| `insecure` | Use for self-signed certificates | No |
| `no-proxy-export` | Prevent Pynt from exporting proxy settings (useful for Selenium/UI tests) | No |

## Outputs

| Output | Description |
|--------|-------------|
| `report-path` | Path to the generated security report |

## Examples

### Example 1: Newman with Multiple Environments

```yaml
name: API Security Testing
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: newman
          pynt-id: ${{ secrets.PYNT_ID }}
          collection: ./postman/collection.json
          environment: ./postman/dev.json,./postman/staging.json
          test-name: "API Security - ${{ github.ref_name }}"
          severity-level: high
          tags: "api,security,newman"
```

### Example 2: Command Mode with Pytest

```yaml
name: API Security Testing
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: command
          pynt-id: ${{ secrets.PYNT_ID }}
          command: "pytest tests/api_tests.py -v"
          test-name: "Pytest API Security Scan"
          application-name: "My API Service"
          severity-level: medium
```

### Example 3: Command Mode with Jest

```yaml
name: API Security Testing
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: command
          pynt-id: ${{ secrets.PYNT_ID }}
          command: "npm test"
          test-name: "Jest API Security Scan"
          report-path: ./pynt-report.json
```

### Example 4: Command Mode with Selenium

```yaml
name: UI and API Security Testing
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install selenium pytest
          pip install -r requirements.txt

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: command
          pynt-id: ${{ secrets.PYNT_ID }}
          command: "pytest tests/selenium_tests.py"
          test-name: "Selenium Security Scan"
          allow-errors: true
```

### Example 5: Newman with mTLS

```yaml
name: API Security Testing with mTLS
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: newman
          pynt-id: ${{ secrets.PYNT_ID }}
          collection: ./postman/collection.json
          tls-client-cert: ./certs/client-cert.pem
          tls-client-key: ./certs/client-key.pem
          host-ca: ./certs/ca.pem
          test-name: "mTLS API Security Scan"
```

### Example 6: RestAssured (Java/Maven)

```yaml
name: API Security Testing
on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run Pynt Security Scan
        uses: pynt-io/api-security-action@v1
        with:
          mode: command
          pynt-id: ${{ secrets.PYNT_ID }}
          command: "mvn test"
          test-name: "RestAssured Security Scan"
```

## Supported Testing Frameworks

The `command` mode works with any testing framework, including:

- **Selenium** - UI testing
- **RestAssured** - Java API testing
- **Jest** - JavaScript testing
- **Pytest** - Python API testing
- **Go** - Go backend testing
- **JMeter** - Performance testing
- **And any other framework** that makes API calls

## Security Notes

- Always store your Pynt ID as a GitHub secret
- Never commit sensitive credentials or tokens to your repository
- Use environment variables for sensitive configuration
- Consider using GitHub's environment protection rules for production scans

## Documentation

- [Pynt Documentation](https://docs.pynt.io)
- [Pynt for Newman](https://docs.pynt.io/documentation/security-testing-integrations/pynt-with-api-testing-clis/pynt-for-newman-postman-cli)
- [Pynt Command Mode](https://docs.pynt.io/documentation/api-security-testing/pynt-cli-modes/pynt-command-cli-mode)
- [Pynt with Testing Frameworks](https://docs.pynt.io/documentation/security-testing-integrations/pynt-with-testing-frameworks)

## Support

For issues, questions, or feature requests, please contact Pynt support or visit [pynt.io](https://pynt.io).

## License

[Add your license here]
