# Mule SonarQube Plugin

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/stn1slv/mulesonarqube)
[![SonarQube Version](https://img.shields.io/badge/SonarQube-9.4+-blue.svg)](https://www.sonarqube.org/)
[![Java Version](https://img.shields.io/badge/Java-17+-orange.svg)](https://openjdk.java.net/)
[![License](https://img.shields.io/badge/License-UNLICENSE-red.svg)](UNLICENSE.md)

A comprehensive SonarQube plugin designed to validate and analyze Mule applications' configuration files. This plugin provides static code analysis capabilities specifically tailored for MuleSoft applications, supporting the Mule 4.x runtime.

## 🚀 Quick Start with Docker

The easiest way to get started is using the pre-configured Docker image:

```bash
# Pull the Docker image
docker pull stn1slv/sonarqube-for-mule:9.9

# Run SonarQube with Mule plugin pre-installed
docker run -d \
  --name sonarqube-mule \
  -p 9000:9000 \
  -p 9092:9092 \
  stn1slv/sonarqube-for-mule:9.9
```

Access SonarQube at `http://localhost:9000` (default credentials: admin/admin)

## 📖 Table of Contents

- [Features](#-features)
- [Quick Start with Docker](#-quick-start-with-docker)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Rules and Metrics](#-rules-and-metrics)
- [Quality Profiles](#-quality-profiles)
- [Development](#-development)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

- **Mule 4.x Support**: Compatible with Mule 4.x applications
- **Comprehensive Rule Set**: 50+ predefined rules covering best practices and common issues
- **Custom Metrics**: Specialized metrics for Mule applications (flows, subflows, transformations)
- **Quality Gates**: Enforce quality standards in your CI/CD pipeline
- **Extensible**: Create custom rules using XPath expressions
- **Coverage Support**: Integration with MUnit test coverage reports
- **Docker Ready**: Pre-configured Docker image available

## 🔧 Installation

### Option 1: Using Docker (Recommended)

Use the pre-built Docker image from [stn1slv/mulesonarqube](https://github.com/stn1slv/mulesonarqube):

```bash
docker pull stn1slv/sonarqube-for-mule:9.9
docker run -d --name sonarqube-mule -p 9000:9000 stn1slv/sonarqube-for-mule:9.9
```

### Option 2: Manual Installation

1. **Build the Plugin**:
   ```bash
   git clone https://github.com/stn1slv/mule-sonarqube-plugin.git
   cd mule-sonarqube-plugin
   mvn clean package
   ```

2. **Install Plugin**:
   - Copy `target/mule-validation-sonarqube-plugin-*.jar` to `<sonar-home>/extensions/plugins/`
   - Copy the `rules-4.xml` rule file for Mule 4.x to `<sonar-home>/extensions/plugins/`

3. **Configure XML Plugin**:
   - Go to Administration → Configuration → General Settings → XML
   - Remove `.xml` extension to prevent conflicts

4. **Restart SonarQube**

This is an [UNLICENSED software, please review the considerations](UNLICENSE.md). 

## 🔧 Configuration

### Project Configuration

#### Quality Profile

The "MuleSoft Rules for Mule 4.x" quality profile is active by default and ready to use.

#### Quality Gate Setup

To enforce custom quality standards:

1. Go to Project → Administration → Quality Gates
2. Select or create a custom quality gate
3. Configure thresholds for metrics like coverage, complexity, and issues

![Quality Gate Configuration](/img/quality-gate.png)

### Maven Configuration

Add to your `pom.xml`:

```xml
<properties>
    <sonar.sources>src/main/mule</sonar.sources>
    <sonar.host.url>http://localhost:9000</sonar.host.url>
    <sonar.language>mule</sonar.language>
</properties>
```

Or configure in `settings.xml`:

```xml
<profile>
    <id>sonar</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <sonar.host.url>http://localhost:9000</sonar.host.url>
    </properties>
</profile>
```

## 🎯 Usage

### Basic Analysis

```bash
# Analyze your Mule project
mvn sonar:sonar

# With custom parameters
mvn sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.sources=src/main/mule \
  -Dsonar.mule.ruleset.categories=flows,configuration,api
```

### Filter Rule Categories

You can specify which rule categories to apply:

```bash
mvn sonar:sonar -Dsonar.mule.ruleset.categories=flows,configuration,api
```

Available categories:
- `application` - Application-level rules
- `flows` - Flow-specific rules  
- `configuration` - Configuration rules
- `api` - API-related rules

### Coverage Integration

For MUnit coverage integration, ensure your `pom.xml` includes:

```xml
<plugin>
    <groupId>com.mulesoft.munit.tools</groupId>
    <artifactId>munit-maven-plugin</artifactId>
    <configuration>
        <coverage>
            <formats>
                <format>json</format>
            </formats>
        </coverage>
    </configuration>
</plugin>
```

## 📊 Rules and Metrics

### Built-in Metrics

| Metric | Description |
|--------|-------------|
| **Flows** | Number of flows in configuration files |
| **SubFlows** | Number of subflows in configuration files |
| **DW Transformations** | Number of DataWeave transformations |
| **Configuration Files** | Number of Mule configuration files |
| **Complexity Rating** | File complexity based on flow count |
| **Coverage** | MUnit test coverage percentage |

### Complexity Rating

- **Simple**: ≤ 7 flows per file
- **Medium**: 8-15 flows per file  
- **Complex**: > 15 flows per file

### Rule Categories

#### Application Rules
- APIKit usage validation
- Exception handling strategy validation
- Security configurations

#### Flow Rules  
- Flow naming conventions
- Flow complexity limits
- Error handling patterns

#### Configuration Rules
- Property placeholder usage
- Secure configuration practices
- DataWeave external file usage
- HTTPS enforcement
- Database connection security

### Custom Rules

Create custom rules by defining XPath expressions in the rule store files:

```xml
<rule id="custom1"
      name="Custom Rule Name"
      description="Custom rule description"
      severity="MAJOR" 
      type="code_smell"
      applies="file">
    count(//mule:flow[@name[not(matches(., '^[a-z][a-zA-Z0-9]*$'))]]) = 0
</rule>
```

## 🏷️ Quality Profiles

The plugin provides a built-in quality profile for Mule 4.x applications.

### Mule 4.x Profile (Default)
- **Name**: "MuleSoft Rules for Mule 4.x"  
- **Rules**: Optimized for Mule 4.x syntax and best practices
- **File**: `rules-4.xml`

![Quality Profiles](/img/quality-profiles.png)

Each profile contains rules organized into categories:

| Category | Description | Example Rules |
|----------|-------------|---------------|
| **Application** | Application-level validations | APIKit usage, Exception strategies |
| **Flows** | Flow-specific rules | Naming conventions, Complexity limits |
| **Configuration** | Global configuration rules | Property usage, Security settings |

## 📈 Analysis Results

### Overview Dashboard
Get a comprehensive view of your Mule application's quality:

![Project Overview](/img/project-overview.png)

### Issues Detection
Identify and track code quality issues:

![Project Issues](/img/project-issues.png)

### Metrics Dashboard
Monitor key metrics specific to Mule applications:

![Project Measures](/img/project-measures.png)

## 🛠️ Development

### Prerequisites
- Java 17+
- Maven 3.6+
- SonarQube 9.4+

### Building from Source

```bash
# Clone the repository
git clone https://github.com/your-username/mule-sonarqube-plugin.git
cd mule-sonarqube-plugin

# Build the plugin
mvn clean package

# The plugin JAR will be generated in target/
```

### Testing

```bash
# Run unit tests
mvn test

# Run integration tests
mvn verify
```

### Creating Custom Rules

1. **Edit Rule Files**: Modify `src/test/resources/rules-4.xml`
2. **Define XPath**: Create XPath expressions to match code patterns
3. **Set Attributes**: Configure severity, type, and scope
4. **Test Rules**: Use the test framework to validate your rules

Example rule structure:
```xml
<rulestore type="mule4">
    <ruleset category="custom">
        <rule id="1"
              name="Custom Rule Name"
              description="Detailed description of the rule"
              severity="MAJOR"
              type="code_smell"
              applies="file"
              locationHint="//mule:flow">
            <!-- XPath expression -->
            count(//mule:flow) &lt; 10
        </rule>
    </ruleset>
</rulestore>
```

### Rule Attributes

| Attribute | Values | Description |
|-----------|--------|-------------|
| **severity** | BLOCKER, CRITICAL, MAJOR, MINOR, INFO | Issue severity level |
| **type** | bug, vulnerability, code_smell | Issue classification |
| **applies** | file, application | Rule scope |
| **locationHint** | XPath expression | Where to highlight issues |

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Add tests for your changes
5. Ensure all tests pass (`mvn test`)
6. Commit your changes (`git commit -m 'Add amazing feature'`)
7. Push to the branch (`git push origin feature/amazing-feature`)
8. Open a Pull Request

### Reporting Issues

Please use the [GitHub Issues](https://github.com/your-username/mule-sonarqube-plugin/issues) page to report bugs or request features.

## 📄 License

This project is released under the [UNLICENSE](UNLICENSE.md). This means the software is in the public domain and you can use it for any purpose without any restrictions.

## 🙏 Acknowledgments

- [SonarQube](https://www.sonarqube.org/) for the excellent code quality platform
- [MuleSoft](https://www.mulesoft.com/) for the Mule runtime and development tools
- [stn1slv](https://github.com/stn1slv) for maintaining the Docker image

## 📞 Support

- **Documentation**: Check this README and inline code documentation
- **Community**: Join discussions in GitHub Issues

---

**Enjoy analyzing your Mule applications and improving code quality! 🚀**

