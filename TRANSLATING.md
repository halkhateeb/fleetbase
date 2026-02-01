# Contributing to Fleetbase Translations

First off, thank you for considering contributing to Fleetbase translations! Your efforts help make Fleetbase accessible to a global audience. This guide will walk you through the process of adding or updating language translations for the Fleetbase platform and its various extensions.

## Understanding the Structure

Fleetbase is a modular system. The main application, known as Fleetbase Console, has its own set of translations. Additionally, each extension (like FleetOps or Storefront) also contains its own translation files. This means that to provide a complete translation for a specific language, you may need to contribute to multiple repositories.

- **Main Application (`fleetbase/fleetbase`)**: Contains the core translation files for the Fleetbase Console.
- **Extensions/Modules**: Each extension has its own repository and its own set of translation files.

## File Format and Location

All translation files are in the **YAML** format (`.yaml` or `.yml`). The base language for all translations is American English (`en-us.yaml`).

- In the main `fleetbase/fleetbase` repository, the translation files are located at `./console/translations/`.
- In each extension repository, the translation files are located at `./translations/`.

Translation files are named using the language and region code, for example:

- `en-us.yaml` (American English)
- `fr-fr.yaml` (French, France)
- `zh-cn.yaml` (Chinese, Simplified)

## How to Contribute Translations

Follow these steps to contribute a new translation or update an existing one.

### Step 1: Fork and Clone the Repository

First, you need to fork the repository you want to contribute to. This could be the main `fleetbase/fleetbase` repository or one of the extension repositories. After forking, clone it to your local machine.

### Step 2: Create or Update a Language File

Navigate to the appropriate translations directory (`./console/translations/` or `./translations/`).

- **To add a new language**: Copy the `en-us.yaml` file and rename it to your target language code (e.g., `es-es.yaml`).
- **To update an existing language**: Open the existing language file. You can compare it with `en-us.yaml` to find missing keys or phrases that need updating.

### Step 3: Translate the Content

Open the YAML file in a text editor. You will see a structure of nested keys and values.

```yaml
# Example from en-us.yaml
common:
  new: New
  create: Create
  delete-selected-count: Delete {count} Selected
```

When translating, you should:

- **Only translate the values**, not the keys. For example, in `new: New`, you would only translate `New`.
- **Keep placeholders intact**. Some phrases contain placeholders like `{count}` or `{resource}`. These should not be translated. They are used by the application to insert dynamic values.

Here is an example of the French translation for the keys above:

```yaml
# Example from fr-fr.yaml
common:
  new: Nouveau
  create: Créer
  delete-selected-count: Supprimer {count} sélectionné(s)
```

### Step 4: Submit a Pull Request

Once you have finished translating, commit your changes and push them to your forked repository. Then, open a pull request to the original Fleetbase repository.

- Make sure your pull request has a clear title and description of the changes you made.
- If you are translating an extension, you may need to submit a pull request to the extension's repository. If your changes also affect the main console, a separate PR to the `fleetbase/fleetbase` repository might be necessary.

Your contribution will be reviewed by the Fleetbase team, and once approved, it will be merged into the project.

## Translation Repositories

Here is a list of the primary repositories that accept translation contributions:

| Repository                               | Translation Path              |
| ---------------------------------------- | ----------------------------- |
| [fleetbase/fleetbase][1]                 | `./console/translations/`     |
| [fleetbase/fleetops][2]                  | `./translations/`             |
| [fleetbase/storefront][3]                | `./translations/`             |
| [fleetbase/dev-engine][4]                | `./translations/`             |
| [fleetbase/iam-engine][5]                | `./translations/`             |
| [fleetbase/pallet][6]                    | `./translations/`             |
| [fleetbase/ledger][7]                    | `./translations/`             |
| [fleetbase/registry-bridge][8]           | `./translations/`             |

[1]: https://github.com/fleetbase/fleetbase
[2]: https://github.com/fleetbase/fleetops
[3]: https://github.com/fleetbase/storefront
[4]: https://github.com/fleetbase/dev-engine
[5]: https://github.com/fleetbase/iam-engine
[6]: https://github.com/fleetbase/pallet
[7]: https://github.com/fleetbase/ledger
[8]: https://github.com/fleetbase/registry-bridge

## Language-Specific Guidelines

### Arabic Translation Guidelines

When translating to Arabic (`ar-ae.yaml`), please keep these best practices in mind:

#### 1. **Formal vs. Informal Language**
- Use formal Arabic (Modern Standard Arabic - MSA) for professional contexts
- Maintain consistency in addressing the user (second person formal)

#### 2. **Technical Terms**
- Some technical terms can be kept in English if commonly used (e.g., "API", "Dashboard")
- When translating technical terms, use widely accepted Arabic equivalents
- Example: "Orders" → "الطلبات" (al-talabat), "Drivers" → "السائقون" (al-sa'iqun)

#### 3. **Text Direction (RTL)**
- Arabic is a right-to-left (RTL) language
- The Fleetbase console automatically applies RTL styling when Arabic is selected
- Numbers, dates, and code snippets will remain left-to-right (LTR) within RTL text

#### 4. **Punctuation**
- Use Arabic punctuation marks: ، (Arabic comma) instead of ,
- Use Arabic question mark: ؟ instead of ?
- Keep parentheses and brackets as they are or mirror them: ) ( becomes ( )

#### 5. **Pluralization**
- Arabic has complex plural rules (singular, dual, plural)
- When using placeholders like `{count}`, ensure the Arabic text accommodates different plural forms
- Example: `delete-selected-count: حذف {count} المحدد` (adjust based on count)

#### 6. **Gender**
- Arabic nouns and adjectives have gender
- Choose the appropriate gender form based on the context
- For generic terms, use masculine form (default in Arabic) or use gender-neutral phrasing when possible

#### 7. **Font Recommendations**
The console uses optimized Arabic fonts automatically when Arabic is selected:
- **Tajawal** - Modern, clean sans-serif
- **Cairo** - Geometric sans-serif
- **Noto Sans Arabic** - Fallback font

#### 8. **Testing Your Translation**
After translating to Arabic:
1. Switch the console to Arabic language
2. Verify that all text displays correctly
3. Check for any text overflow or layout issues
4. Ensure buttons and UI elements have enough space
5. Verify that RTL layout works correctly in all pages

### Persian (Farsi) Translation Guidelines

Similar to Arabic, Persian (`fa-ir.yaml`) is also RTL. Follow similar guidelines with these additions:

1. **Persian-Specific Characters**: Use Persian numbers (۰۱۲۳۴۵۶۷۸۹) where appropriate
2. **Vocabulary**: Persian has different vocabulary than Arabic despite using the same script
3. **Font**: The console uses Vazir and Samim fonts optimized for Persian

### Chinese Translation Guidelines

When translating to Chinese (`zh-cn.yaml`):

1. **Simplified vs. Traditional**: Use Simplified Chinese for `zh-cn`
2. **Tone**: Maintain professional and concise language
3. **Technical Terms**: Many technical terms are transliterated or kept in English
4. **Length**: Chinese text is typically shorter than English, adjust UI accordingly

### Spanish Translation Guidelines

We support both European Spanish (`es-es.yaml`) and Mexican Spanish (`es-mx.yaml`):

1. **Regional Differences**: 
   - Spain Spanish uses "vosotros" form
   - Mexican Spanish uses "ustedes" form
2. **Vocabulary**: Some words differ between regions (e.g., "ordenador" vs "computadora")
3. **Formality**: Maintain formal register using "usted" form

## Translation Quality Checklist

Before submitting your translation, ensure:

- [ ] All keys from `en-us.yaml` are present
- [ ] No translation values are empty (unless intentionally left blank)
- [ ] Placeholders like `{count}`, `{name}` are preserved
- [ ] Text fits in UI elements (test if possible)
- [ ] Consistent terminology throughout
- [ ] Grammar and spelling are correct
- [ ] Punctuation follows language conventions
- [ ] No machine translation artifacts
- [ ] Technical terms are accurate
- [ ] Context-appropriate formality level

## Translation Tools

### Recommended Tools
- **YAML Editor**: Use a proper YAML editor to avoid formatting errors
- **CAT Tools**: Consider using translation memory tools for consistency
- **Spell Checkers**: Use language-specific spell checkers

### Validation
Run the translation linter before submitting:

```bash
cd console
npm run lint:translations
```

This will check for:
- Missing keys
- Invalid YAML syntax
- Malformed placeholders
- Encoding issues

Thank you again for your contribution to the Fleetbase community!
