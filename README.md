# 🪵🛒 Lumber Mart

A plugin marketplace for Claude Code that lets you discover and install plugins for enhanced development workflows.

Learn more about Claude Code marketplaces in the [official documentation](https://code.claude.com/docs/en/plugin-marketplaces).

## Available Plugins

### wavejson-plugin

Generate WaveJSON timing diagrams for digital signals and create HTML viewers to display them using WaveDrom. Perfect for hardware development, protocol analysis, and documenting signal timing in embedded systems projects.

**Features:**
- Generate WaveJSON format timing diagrams
- Create standalone HTML viewers with WaveDrom rendering
- Support for clock signals, data buses, control signals, and protocol sequences
- Built-in templates for common patterns (SPI, I2C, setup/hold timing, etc.)
- Quick reference documentation included

### circuit-synth-plugin

Create and design PCB circuits using Python and KiCad with AI assistance. Generate circuits from natural language descriptions, existing documentation, or templates. Search components, manage BOMs, and produce manufacturing-ready outputs.

**Features:**
- Define circuits in Python instead of GUI clicking
- Generate professional KiCad output (.kicad_pro, .kicad_sch, .kicad_pcb)
- AI-accelerated design workflows from natural language descriptions
- Convert hardware documentation/pinouts into formal schematics
- Search components with real-time stock and pricing (JLCPCB, DigiKey, Mouser)
- Pre-built manufacturing-ready patterns (buck/boost converters, LDO regulators, etc.)
- Generate manufacturing outputs (BOM, Gerbers, PDF schematics)
- Hierarchical design with modular subcircuits
- Version control friendly (text-based definitions)

## Usage

### Adding This Marketplace

```bash
# If hosted on GitHub
/plugin marketplace add lumberbarons/lumber-mart

# For local testing
/plugin marketplace add /path/to/lumber-mart
```

### Installing Plugins

```bash
# List available plugins
/plugin list

# Install a plugin
/plugin install wavejson-plugin@lumber-mart
/plugin install circuit-synth-plugin@lumber-mart
```

## License

MIT
