# How to Generate PowerPoint Presentation

This guide explains different methods to convert the Markdown presentation to PowerPoint format.

---

## Method 1: Marp (Recommended - Best Formatting)

Marp is a Markdown presentation ecosystem that creates beautiful slides.

### Installation:
```bash
npm install -g @marp-team/marp-cli
```

### Generate PPTX:
```bash
marp Salesforce_OAuth_Security_Presentation.md --pptx -o Salesforce_OAuth_Security.pptx
```

### Generate PDF:
```bash
marp Salesforce_OAuth_Security_Presentation.md --pdf -o Salesforce_OAuth_Security.pdf
```

### Generate HTML (Interactive):
```bash
marp Salesforce_OAuth_Security_Presentation.md -o presentation.html
```

### With Custom Theme:
```bash
marp Salesforce_OAuth_Security_Presentation.md --theme default --pptx
```

Available themes: `default`, `gaia`, `uncover`

---

## Method 2: Pandoc (Universal Converter)

Pandoc is a universal document converter.

### Installation (macOS):
```bash
brew install pandoc
```

### Generate PPTX:
```bash
pandoc Salesforce_OAuth_Security_Presentation.md -o Salesforce_OAuth_Security.pptx
```

### With Custom Reference Template:
```bash
pandoc Salesforce_OAuth_Security_Presentation.md \
  --reference-doc=template.pptx \
  -o Salesforce_OAuth_Security.pptx
```

---

## Method 3: Reveal.js (Web-Based Presentation)

Create an interactive HTML presentation.

### Using Pandoc:
```bash
pandoc Salesforce_OAuth_Security_Presentation.md \
  -t revealjs \
  -s \
  -o presentation.html
```

### Standalone Reveal.js:
```bash
# Clone reveal.js
git clone https://github.com/hakimel/reveal.js.git
cd reveal.js
npm install
npm start
# Then copy your markdown content into a reveal.js template
```

---

## Method 4: Slidev (Modern Vue-based)

Slidev is a developer-friendly presentation tool.

### Installation:
```bash
npm init slidev
```

Then copy the markdown content and customize.

---

## Method 5: Manual Copy-Paste

### To PowerPoint/Keynote/Google Slides:
1. Open PowerPoint/Keynote/Google Slides
2. Each `---` separator marks a new slide
3. Copy slide content section by section
4. Format as needed with your company template

### To Prezi:
1. Go to prezi.com
2. Create new presentation
3. Copy sections from the markdown
4. Use Prezi's visual canvas to arrange content

---

## Quick Comparison

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| **Marp** | Easy, good formatting, multiple outputs | Limited customization | Quick professional decks |
| **Pandoc** | Universal, flexible | Basic formatting | Simple conversions |
| **Reveal.js** | Interactive, web-based, impressive | Requires hosting | Tech presentations |
| **Slidev** | Modern, developer-friendly | Learning curve | Developer audiences |
| **Manual** | Full control, custom branding | Time-consuming | Corporate templates |

---

## Recommended Workflow

### For Quick Results:
```bash
# Install Marp
npm install -g @marp-team/marp-cli

# Generate PPTX
marp Salesforce_OAuth_Security_Presentation.md --pptx
```

### For Custom Styling:
1. Generate initial PPTX with Marp/Pandoc
2. Open in PowerPoint
3. Apply your company template
4. Customize colors, fonts, logos

### For Interactive Presentations:
```bash
# Generate HTML with Marp
marp Salesforce_OAuth_Security_Presentation.md -o presentation.html

# Open in browser and present
open presentation.html
```

---

## Tips

- **Keep source MD file:** Always version control the markdown source
- **Don't commit generated files:** Add `*.pptx`, `*.pdf` to `.gitignore`
- **Use consistent styling:** Choose one method and stick with it
- **Test before presenting:** Always review the generated slides
- **Have a backup:** Keep both MD and PPTX versions ready

---

## Troubleshooting

### Marp: Command not found
```bash
# Check installation
npm list -g @marp-team/marp-cli

# Reinstall if needed
npm install -g @marp-team/marp-cli
```

### Pandoc: Installation issues on macOS
```bash
# Alternative installation
brew update
brew install pandoc

# Or download from: https://pandoc.org/installing.html
```

### Diagrams not rendering
- ASCII diagrams may not render perfectly in PPTX
- Consider converting to images for critical diagrams
- Use Mermaid or PlantUML for complex diagrams

---

## Additional Resources

- [Marp Documentation](https://marp.app/)
- [Pandoc User Guide](https://pandoc.org/MANUAL.html)
- [Reveal.js](https://revealjs.com/)
- [Slidev](https://sli.dev/)
