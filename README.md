# Claude Code Skills Collection

English | [한국어](README_KR.md)

A curated collection of production-ready skills for [Claude Code](https://github.com/anthropics/claude-code) that enhance your development workflow with AI-powered automation.

## 📦 Available Skills

This repository is organized as a **monorepo** containing multiple Claude Code skills, each designed to solve specific development challenges:

### 🔄 [change-log](skills/change-log/)
**Automated Changelog Generation for Jira & Confluence**

Automatically generates comprehensive change logs from your git branches and publishes them to Confluence with full Jira integration.

**Key Features:**
- 🎯 Automatic Jira ticket detection from branch names
- 📊 AI-powered git diff analysis
- 📝 One-click Confluence documentation
- 🔄 Smart page management (create or append)
- 🤖 Intelligent impact analysis and technical summaries

**Use cases:** Release documentation, team collaboration, change tracking

[→ View full documentation](skills/change-log/)

---

### 🛠️ [api-codegen](skills/api-codegen/)
**Production-Ready API Client Generator**

Generate type-safe, production-ready API client code from Swagger/OpenAPI specifications with interactive customization.

**Key Features:**
- 📋 Parse Swagger/OpenAPI (URL or local file)
- 🔍 Analyze existing project structure and code style
- 🎨 Generate code matching your project conventions
- ✅ Create comprehensive unit and integration tests
- 🔧 Support for multiple languages (Kotlin, Java, TypeScript, Python)
- 🏗️ Framework-aware (Spring Boot, React, Vue, FastAPI, etc.)

**Use cases:** Microservice integration, third-party API consumption, backend-frontend alignment

[→ View full documentation](skills/api-codegen/)

---

## 🚀 Quick Start

### Installation via Marketplace

1. **Add the marketplace:**
```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

2. **Install a skill:**
```bash
# Install changelog generator
/plugin install change-log

# Or install API code generator
/plugin install api-codegen
```

3. **Use it in your next conversation:**
```bash
/change-log
# or
/api-codegen https://api.example.com/swagger.json
```

### Verify Installation

Ask Claude to list available skills:
```
What skills are available?
```

You should see the installed skills in the response.

## 📖 How to Use

Each skill comes with its own comprehensive documentation:
- [change-log Usage Guide](skills/change-log/)
- [api-codegen Usage Guide](skills/api-codegen/)

Basic usage pattern:
```bash
# Via skill command
/skill-name [options]

# Or via natural language
"generate a changelog"
"create API client from swagger"
```

## 🗂️ Repository Structure

```
.
├── skills/
│   ├── change-log/                    # Changelog generation plugin
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json            # Plugin manifest
│   │   └── SKILL.md                   # Skill prompt & documentation
│   └── api-codegen/                   # API code generator plugin
│       ├── .claude-plugin/
│       │   └── plugin.json            # Plugin manifest
│       └── SKILL.md                   # Skill prompt & documentation
├── .claude-plugin/
│   └── marketplace.json               # Marketplace configuration
└── README.md                          # This file
```

Each plugin contains:
- **`.claude-plugin/plugin.json`**: Plugin metadata (name, version, description, author)
- **`SKILL.md`**: The actual skill prompt and detailed documentation

## 🔮 Roadmap & Future Skills

We're continuously expanding this collection with new productivity-boosting skills. Planned additions include:

- 🧪 **test-generator**: Intelligent test generation from existing code
- 📚 **doc-sync**: Keep documentation in sync with code changes
- 🔐 **security-audit**: Automated security vulnerability scanning
- 🎯 **code-reviewer**: AI-powered code review and suggestions
- 🔄 **migration-helper**: Assist with framework/library migrations

*Have an idea for a new skill?* [Open an issue](https://github.com/kdanuu/tna-plugin-marketplace/issues) or submit a pull request!

## 🤝 Contributing

We welcome contributions! Here's how you can help:

1. **Report bugs or request features** via [GitHub Issues](https://github.com/kdanuu/tna-plugin-marketplace/issues)
2. **Submit improvements** through pull requests
3. **Share your own skills** - we'd love to include them!

### Adding a New Skill

1. Create a new directory under `skills/`
2. Add a `SKILL.md` file with your skill prompt
3. Update `.claude-plugin/marketplace.json`
4. Test your skill thoroughly
5. Submit a pull request

See existing skills for reference structure.

## 📋 Requirements

- [Claude Code CLI](https://github.com/anthropics/claude-code) (latest version recommended)
- Git (for version control features)
- Additional requirements vary by skill (see individual skill documentation)

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details

## 👤 Author

Created and maintained by **danwoo-kim** ([@kdanuu](https://github.com/kdanuu))

## 🌟 Support

If you find these skills helpful, please:
- ⭐ Star this repository
- 🐛 Report issues you encounter
- 💡 Suggest new features or skills
- 📢 Share with your team

---

**Happy Coding with Claude!** 🎉
