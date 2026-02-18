---
name: drupal-recipe-content
description: Create Drupal recipes that import content entities (taxonomy terms, nodes, media, menu links) with multilingual translations. Use this skill whenever the user wants to create a Drupal recipe with default content, import taxonomy terms or other content entities into Drupal via recipes, set up multilingual/translated content in a recipe, or export existing content for use in a recipe. Also trigger when the user mentions "default content in recipes", "recipe with translations", or "content export for recipe".
---

# Drupal Recipe Content Import

Create Drupal recipes that ship default content (taxonomy terms, nodes, media, etc.) with full multilingual translation support, using Drupal core's built-in content export/import API.

## Key Concepts

- Drupal recipes are primarily for **configuration**, but since **Drupal 11.3**, core includes a content export/import API that allows recipes to ship content entities.
- Content is stored as YAML files in a `content/` directory inside the recipe.
- Translations are included in the exported YAML — no separate files needed per language.
- On import, entities get **new IDs** (not the original ones). UUIDs are used internally for dependency resolution.
- The `default_content` contrib module is **unmaintained and conflicts with Canvas** — do not use it. Use core's built-in API instead.

## Recipe Structure

```
my_recipe/
├── recipe.yml
├── config/
│   └── ... (optional config YAML files)
└── content/
    └── taxonomy_term/
        ├── <uuid-1>.yml
        ├── <uuid-2>.yml
        └── ...
```

Content files are organized by entity type in subdirectories under `content/`. Each entity is a separate YAML file named by its UUID.

## recipe.yml

The `recipe.yml` does not need to explicitly reference content files. Drupal automatically imports everything in the `content/` directory when the recipe is applied.

```yaml
name: 'My Taxonomy Terms'
description: 'Provides taxonomy terms with EN/DE translations.'
type: 'Content'

recipes:
  # Ensure the vocabulary and translation config exist first
  - core/recipes/tags_taxonomy

install:
  - language
  - content_translation
```

Make sure any required vocabulary, content type, or field configuration is applied **before** the content import — either via dependent recipes or config in the same recipe.

## Exporting Content

### Drupal 11.3+ (Core CLI)

```bash
# Export a single taxonomy term
php core/scripts/drupal content:export taxonomy_term 42

# Export with all dependencies (referenced entities, files, etc.)
php core/scripts/drupal content:export taxonomy_term 42 --with-dependencies --dir=recipes/my_recipe/content

# Export a node with dependencies
php core/scripts/drupal content:export node 3 -W --dir=recipes/my_recipe/content
```

The `-W` / `--with-dependencies` flag is important — it ensures referenced entities (images, terms, users) are exported alongside the main entity.

### Drush (with default_content as dev-only dependency)

If using Drush, `default_content` can be a **dev-only** dependency (not needed in production):

```bash
composer require --dev drupal/default_content
drush en -y default_content

# Export to a recipe's content directory
drush dcer taxonomy_term 42 --folder=recipes/my_recipe/content

# Export all terms of a vocabulary (if supported)
drush dcer taxonomy_term --folder=recipes/my_recipe/content
```

## Multilingual Content

Translations are embedded in the exported YAML automatically. When you export an entity that has translations, the YAML will contain all language versions.

### Workflow for multilingual recipes

1. Create content on a reference site with all translations in place
2. Export with `--with-dependencies` — translations are included
3. Place the YAML files in `content/<entity_type>/` inside your recipe
4. When the recipe is applied, entities are created with all their translations

There is no need for separate export/import steps per language.

## Gotchas and Tips

- **IDs change on import**: Exported entity IDs (tid, nid) will not match on the target site. Drupal assigns new sequential IDs. References between entities are resolved via UUIDs.
- **Config must exist first**: The vocabulary, content type, fields, and language configuration must be in place before content is imported. Use the `recipes:` and `install:` sections to ensure this.
- **No `default_content` at runtime**: If you used Drush's `dcer` command for exporting, you only need `default_content` as a dev dependency. The core import mechanism handles the rest.
- **File assets**: When exporting with `--with-dependencies`, file entities and their physical files are included in the export. They get placed in a `file/` subdirectory under `content/`.
- **Entity references**: Cross-references between content entities (e.g., a node referencing taxonomy terms) are resolved by UUID during import, so export order doesn't matter.
