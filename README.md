# Salesforce OAuth Security Guide

A comprehensive guide to Salesforce OAuth 2.0 authentication covering all three flows with security best practices, detailed documentation, and working implementation examples.

## 📚 Contents

- **Authentication vs Authorization** - Clear explanations with real-world analogies
- **Multi-Factor Authentication (MFA)** - Why and how
- **OAuth Access Tokens** - Components and lifecycle
- **Session Tracking & Governance** - Security accountability
- **Three OAuth Flows:**
  - Web Server Flow (user-interactive)
  - JWT Bearer Flow (server-to-server)
  - Client Credentials Flow (machine-to-machine)
- **Security Best Practices**
- **Implementation examples in Python and Node.js**

## 🎯 Target Audience

- Salesforce Administrators
- Security Teams
- Integration Developers
- Solution Architects
- Technical Managers

## 📖 Documentation

### Security & Overview
- **[Salesforce OAuth Security Presentation](Salesforce_OAuth_Security_Presentation.md)** - Comprehensive security presentation with hotel analogies
- [HOW_TO_GENERATE_PPTX.md](HOW_TO_GENERATE_PPTX.md) - Convert presentation to PowerPoint

### OAuth Flow Details
- [Web Server Flow](docs/oauth-flows/oauth_web_server_flow.md) - Interactive user authentication
- [JWT Bearer Flow](docs/oauth-flows/oauth_jwt_flow.md) - Certificate-based server authentication
- [Client Credentials Flow](docs/oauth-flows/oauth_client_credentials_flow.md) - Machine-to-machine authentication

### Implementation Guides
- [Demo Code Documentation](README_DEMOS.md) - Setup and usage for Python & Node.js examples

## 🚀 Quick Start

### Generate PowerPoint (Recommended):
```bash
# Install Marp
npm install -g @marp-team/marp-cli

# Generate PPTX
marp Salesforce_OAuth_Security_Presentation.md --pptx
```

### Alternative Methods:
See [HOW_TO_GENERATE_PPTX.md](HOW_TO_GENERATE_PPTX.md) for more options including:
- Pandoc conversion
- Reveal.js web presentations
- Manual import to PowerPoint/Prezi

## 📝 Customization

The markdown source can be easily customized:
- Add your company branding
- Include specific use cases
- Adjust technical depth
- Add or remove slides

## 🎨 Presentation Features

- 25 comprehensive slides
- Visual diagrams for each OAuth flow
- Hotel analogies throughout for easy understanding
- Comparison tables
- Security best practices checklist
- Real-world scenarios

## 📄 License

Feel free to use and customize this presentation for your organization's training needs.

## 🤝 Contributing

Suggestions and improvements are welcome! Feel free to:
- Submit issues for errors or improvements
- Propose additional content
- Share feedback

## ✨ Key Concepts Covered

1. **Authentication** - Proving identity (check-in at hotel)
2. **Authorization** - Access permissions (room key privileges)
3. **OAuth Tokens** - Temporary credentials with metadata
4. **Session Management** - Tracking and accountability
5. **Security Governance** - Best practices and compliance

---

**Created:** January 2026
**Version:** 1.0
