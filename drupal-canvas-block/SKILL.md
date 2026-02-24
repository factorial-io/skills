---
name: drupal-canvas-block
description: Create Canvas-compatible Drupal custom blocks. Use when the user wants to create a custom block that integrates with Drupal Canvas page builder, display entity data in blocks, or needs blocks that work in both Canvas editor and live pages.
---

# Drupal Canvas Block Creator

You are an expert at creating custom Drupal blocks that integrate seamlessly with Canvas page builder.

## Important Context

**Canvas 1.1 does not support paragraphs directly.** When you need to display paragraph data in Canvas templates, you must create a custom block as a bridge. This skill teaches the pattern for creating Canvas-compatible blocks that retrieve and render paragraph data.

## When to Use This Skill

Use this skill when the user needs to:
- **Display paragraphs in Canvas** (Canvas 1.1+ doesn't support paragraphs natively)
- Create a custom block that displays data from entities (nodes, paragraphs, etc.)
- Make a block work with Canvas page builder
- Create reusable content display blocks for Canvas templates
- Work around Canvas limitations with entity references

## Canvas Block Architecture

### Why Custom Blocks for Paragraphs?

**Canvas 1.1 Limitation:** Canvas page builder cannot directly display paragraph fields. When your content type has paragraph fields (entity reference revisions), Canvas cannot render them in templates.

**Solution:** Create a custom block that acts as a bridge:
- Block retrieves paragraph data from the parent entity
- Block processes and renders the paragraphs
- Canvas adds the block to templates (blocks ARE supported)

### Architecture Pattern

Canvas-compatible blocks for paragraphs follow this pattern:

```
Content Entity (node/taxonomy/media)
    ↓ (has paragraph field)
Paragraph Entities (field_value, field_label, field_media, etc.)
    ↓ (retrieved by)
Custom Drupal Block (Plugin\Block)
    ↓ (processes data)
Block Template (Twig)
    ↓ (renders using)
SDC Components (individual item display)
```

**Example:** Profession node → field_facts (paragraphs) → Facts Block → Fact Component

## Step-by-Step Implementation

### Step 1: Analyze Requirements

**Ask the user:**
1. What entity type will this block display data from? (node, taxonomy, media, etc.)
2. What fields/data should be displayed?
3. Should it work in Canvas editor preview or just on live pages?
4. Should the heading/title be customizable per entity?

### Step 2: Create the Block Plugin

**File:** `web/modules/custom/{module_name}/src/Plugin/Block/{BlockName}.php`

**Template for Canvas-compatible block:**

```php
<?php

declare(strict_types=1);

namespace Drupal\{module_name}\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Plugin\ContainerFactoryPluginInterface;
use Drupal\Core\Routing\RouteMatchInterface;
use Drupal\{entity_type}\{EntityTypeInterface};
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Provides a {description} block.
 *
 * @Block(
 *   id = "{module}_{block_id}",
 *   admin_label = @Translation("{Block Label}"),
 *   category = @Translation("{Category}")
 * )
 */
final class {BlockName} extends BlockBase implements ContainerFactoryPluginInterface {

  /**
   * Constructs a new {BlockName} instance.
   */
  public function __construct(
    array $configuration,
    $plugin_id,
    $plugin_definition,
    private readonly RouteMatchInterface $routeMatch,
  ) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition): self {
    return new self(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('current_route_match')
    );
  }

  /**
   * {@inheritdoc}
   */
  public function defaultConfiguration(): array {
    return [
      'label_display' => FALSE,
    ];
  }

  /**
   * {@inheritdoc}
   */
  public function blockForm($form, FormStateInterface $form_state): array {
    $form = parent::blockForm($form, $form_state);
    // Add custom configuration fields here if needed.
    // Keep minimal to avoid Canvas validation errors.
    return $form;
  }

  /**
   * Gets the current entity from route.
   *
   * @return \Drupal\{entity_type}\{EntityTypeInterface}|null
   *   The entity if available, NULL otherwise.
   */
  private function getCurrentEntity(): ?{EntityTypeInterface} {
    $entity = $this->routeMatch->getParameter('{entity_type}');
    return $entity instanceof {EntityTypeInterface} ? $entity : NULL;
  }

  /**
   * {@inheritdoc}
   */
  public function build(): array {
    $entity = $this->getCurrentEntity();

    // Show Canvas preview message if no proper context.
    if (!$entity || $entity->bundle() !== '{target_bundle}') {
      return [
        '#markup' => '<div style="padding: 2rem; border: 2px dashed #ccc; border-radius: 8px; text-align: center; background: #f9f9f9;">
          <p style="margin: 0; color: #666; font-size: 14px;">
            <strong>{Block Name}</strong><br>
            {Description of what this block displays}
          </p>
        </div>',
      ];
    }

    // Extract and process data from entity fields.
    $data = $this->extractData($entity);

    if (empty($data)) {
      return [];
    }

    return [
      '#theme' => '{module}_{block_template}',
      '#data' => $data,
    ];
  }

  /**
   * Extracts data from the entity.
   *
   * @param \Drupal\{entity_type}\{EntityTypeInterface} $entity
   *   The entity to extract data from.
   *
   * @return array
   *   Processed data array.
   */
  private function extractData({EntityTypeInterface} $entity): array {
    $data = [];

    // Example: Extract paragraph data
    if ($entity->hasField('field_items')) {
      foreach ($entity->get('field_items')->referencedEntities() as $paragraph) {
        $item = [];

        // Extract fields from paragraph
        if ($paragraph->hasField('field_value') && !$paragraph->get('field_value')->isEmpty()) {
          $item['value'] = $paragraph->get('field_value')->value;
        }

        // Add to data array if valid
        if (!empty($item)) {
          $data[] = $item;
        }
      }
    }

    return $data;
  }

  /**
   * {@inheritdoc}
   */
  public function getCacheContexts(): array {
    return array_merge(parent::getCacheContexts(), ['route']);
  }

  /**
   * {@inheritdoc}
   */
  public function getCacheTags(): array {
    $tags = parent::getCacheTags();

    $entity = $this->getCurrentEntity();
    if ($entity) {
      $tags = array_merge($tags, $entity->getCacheTags());

      // Add cache tags for all referenced entities.
      if ($entity->hasField('field_items')) {
        foreach ($entity->get('field_items')->referencedEntities() as $referenced) {
          $tags = array_merge($tags, $referenced->getCacheTags());

          // Add nested references (e.g., media in paragraphs)
          if ($referenced->hasField('field_media') && !$referenced->get('field_media')->isEmpty()) {
            $media = $referenced->get('field_media')->entity;
            if ($media) {
              $tags = array_merge($tags, $media->getCacheTags());
            }
          }
        }
      }
    }

    return $tags;
  }

}
```

### Step 3: Add Theme Hook

**File:** `web/modules/custom/{module_name}/{module_name}.module`

```php
/**
 * Implements hook_theme().
 */
function {module_name}_theme(): array {
  return [
    '{module}_{block_template}' => [
      'variables' => [
        'data' => [],
        'heading' => NULL,
        // Add more variables as needed
      ],
      'template' => '{module}-{block-template}',
    ],
  ];
}
```

### Step 4: Create Block Template

**File:** `web/modules/custom/{module_name}/templates/{module}-{block-template}.html.twig`

**Best practices for Canvas-compatible templates:**

```twig
{#
/**
 * @file
 * Template for {block description}.
 *
 * Available variables:
 * - data: Array of items to display
 * - heading: Section heading (optional)
 */
#}
<section class="container mx-auto px-4 py-12">
  {% if heading %}
    <div class="mb-8">
      {% include 'theme_name:heading' with {
        heading_text: heading,
        level: 2,
      } only %}
    </div>
  {% endif %}

  {% if data %}
    <div class="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {% for item in data %}
        {# Use SDC components when possible #}
        {% include 'theme_name:component' with item only %}
      {% endfor %}
    </div>
  {% endif %}
</section>
```

### Step 5: Canvas Preview Support

**Always provide a helpful preview message** for Canvas editor:

```php
// In build() method, before processing data
if (!$entity || $entity->bundle() !== 'target_bundle') {
  return [
    '#markup' => '<div style="padding: 2rem; border: 2px dashed #ccc; border-radius: 8px; text-align: center; background: #f9f9f9;">
      <p style="margin: 0; color: #666; font-size: 14px;">
        <strong>Block Name</strong><br>
        Clear description of what this block displays and where the data comes from.
      </p>
    </div>',
  ];
}
```

**Why this matters:**
- Canvas editor may not have proper entity context
- Preview message provides clear feedback to content editors
- Prevents confusing empty states

## Canvas Integration Checklist

### ✅ Block Annotation Requirements

```php
/**
 * @Block(
 *   id = "unique_block_id",
 *   admin_label = @Translation("Human Readable Label"),
 *   category = @Translation("Category Name")
 * )
 */
```

**DO NOT add:**
- ❌ `context_definitions` - causes Canvas JavaScript errors
- ❌ Complex configuration - keep it minimal

### ✅ Configuration Best Practices

**For Canvas compatibility:**

1. **Keep configuration minimal** - Only add fields you absolutely need
2. **Always include `blockForm()`** - Even if empty, Canvas needs it:
   ```php
   public function blockForm($form, FormStateInterface $form_state): array {
     $form = parent::blockForm($form, $form_state);
     return $form;
   }
   ```
3. **Avoid custom configuration for dynamic data** - Use entity fields instead
4. **Set `label_display` to FALSE** - Usually blocks in Canvas don't need labels

### ✅ Data Extraction Pattern

**Always use a dedicated method for data extraction:**

```php
private function extractData(EntityInterface $entity): array {
  $data = [];

  // Check field exists and is not empty
  if ($entity->hasField('field_items') && !$entity->get('field_items')->isEmpty()) {
    foreach ($entity->get('field_items')->referencedEntities() as $item) {
      // Extract and validate data
      $processed = $this->processItem($item);
      if ($processed) {
        $data[] = $processed;
      }
    }
  }

  return $data;
}
```

**Benefits:**
- Easier to test
- Cleaner build() method
- Reusable logic

### ✅ Cache Handling

**CRITICAL: Include cache tags for ALL referenced entities:**

```php
public function getCacheTags(): array {
  $tags = parent::getCacheTags();

  $entity = $this->getCurrentEntity();
  if ($entity) {
    // Entity cache tag
    $tags = array_merge($tags, $entity->getCacheTags());

    // Referenced entities (paragraphs, media, etc.)
    foreach ($entity->get('field_items')->referencedEntities() as $item) {
      $tags = array_merge($tags, $item->getCacheTags());

      // Nested references (media in paragraphs, etc.)
      if ($item->hasField('field_media') && !$item->get('field_media')->isEmpty()) {
        $media = $item->get('field_media')->entity;
        if ($media) {
          $tags = array_merge($tags, $media->getCacheTags());
        }
      }
    }
  }

  return $tags;
}
```

**Why this matters:**
- Without proper cache tags, blocks show stale data after entity updates
- Must invalidate when ANY referenced entity changes
- Includes paragraphs, media, taxonomy terms, etc.

### ✅ Route Context

**Use route for entity context**, not Canvas context:

```php
private function getCurrentEntity(): ?EntityInterface {
  $entity = $this->routeMatch->getParameter('entity_type_id');
  return $entity instanceof EntityInterface ? $entity : NULL;
}
```

**DON'T use:**
```php
// ❌ Causes Canvas errors
$entity = $this->getContextValue('entity');
```

## Common Pitfalls & Solutions

### Issue 1: Canvas JavaScript Error "Cannot read properties of undefined (reading 'source')"

**Cause:** Block has `context_definitions` in annotation or missing `blockForm()`

**Solution:**
- Remove `context_definitions` from block annotation
- Add minimal `blockForm()` method (even if empty)
- Add `defaultConfiguration()` with at least `label_display`

### Issue 2: Block Shows Nothing in Canvas

**Cause:** Block returns empty array `[]` when entity context not available

**Solution:** Always return preview markup:
```php
if (!$entity) {
  return ['#markup' => '<div>Preview message...</div>'];
}
```

### Issue 3: Stale Data After Content Updates

**Cause:** Missing cache tags for referenced entities

**Solution:** Add cache tags for ALL referenced entities:
- Main entity
- Paragraphs
- Media entities
- Taxonomy terms
- Any nested references

### Issue 4: PHPStan Error "Property never read, only written"

**Cause:** Injected service not actually used

**Solution:** Remove unused dependencies from constructor

### Issue 5: Canvas Validation Error for Custom Configuration

**Cause:** Block configuration fields that Canvas doesn't recognize

**Solution:**
- Don't use block configuration for dynamic data
- Instead, add fields to the entity and read them in the block
- Keep block configuration minimal (standard Drupal settings only)

## Best Practices for Dynamic Headings

**DON'T** make headings block configuration fields - Canvas can't map them dynamically.

**DO** add heading fields to the entity:

```yaml
# config/sync/field.field.node.{bundle}.field_{section}_heading.yml
field_name: field_{section}_heading
entity_type: node
bundle: {bundle}
label: '{Section} heading'
field_type: string
default_value:
  - value: 'Default Heading'
translatable: true
```

**Then read in block:**
```php
$heading = $this->t('Default');
if ($entity->hasField('field_{section}_heading') && !$entity->get('field_{section}_heading')->isEmpty()) {
  $heading = $entity->get('field_{section}_heading')->value;
}
```

**Benefits:**
- Content editors can customize per entity
- Translatable
- No Canvas configuration issues
- Proper cache invalidation

## Working with Paragraphs

Paragraphs are common in Canvas blocks since Canvas can't display them directly.

### Example: Facts Block Pattern

```php
// Extract paragraph data
$items = [];
$paragraphs = $entity->get('field_items')->referencedEntities();

foreach ($paragraphs as $paragraph) {
  $item = [];

  // Text fields
  if ($paragraph->hasField('field_value') && !$paragraph->get('field_value')->isEmpty()) {
    $item['value'] = $paragraph->get('field_value')->value;
  }

  // Media/image fields
  if ($paragraph->hasField('field_icon') && !$paragraph->get('field_icon')->isEmpty()) {
    $media = $paragraph->get('field_icon')->entity;
    if ($media && $media->hasField('field_media_svg_file')) {
      $file = $media->get('field_media_svg_file')->entity;
      if ($file) {
        $item['icon'] = [
          'src' => $file->createFileUrl(FALSE),
          'width' => 32,
          'height' => 32,
          'alt' => $media->label() ?? '',
        ];
      }
    }
  }

  // Only add if has required data
  if (!empty($item['value'])) {
    $items[] = $item;
  }
}
```

### SVG Media Handling

For SVG icons in paragraphs:

```php
// SVG media typically uses field_media_svg_file
if ($media && $media->hasField('field_media_svg_file')) {
  $file = $media->get('field_media_svg_file')->entity;
  if ($file) {
    $icon_data = [
      'src' => $file->createFileUrl(FALSE), // Relative URL
      'width' => 32,
      'height' => 32,
      'alt' => $media->label() ?? '',
    ];
  }
}
```

## Template Best Practices

### Conditional Icon Rendering

When passing data to SDC components that might not have all fields:

```twig
{# DON'T pass undefined keys - causes Canvas validation errors #}
{% for item in items %}
  {# Build props conditionally #}
  {% if item.icon is defined and item.icon is not empty %}
    {% include 'theme:component' with {
      value: item.value,
      label: item.label,
      icon: item.icon,
    } only %}
  {% else %}
    {% include 'theme:component' with {
      value: item.value,
      label: item.label,
    } only %}
  {% endif %}
{% endfor %}
```

### Using SDC Components from Block Templates

```twig
{# Render individual items using SDC components #}
{% for item in items %}
  {% include 'theme_name:component_name' with item only %}
{% endfor %}
```

**Tips:**
- Use `only` to prevent variable bleeding
- Component handles its own conditionals
- Keep template logic minimal

## Testing Checklist

### Before Committing:

1. **Test on live page:**
   - Visit entity page (e.g., `/node/1`)
   - Verify block displays correctly
   - Check all data renders properly

2. **Test in Canvas editor:**
   - Go to Canvas template editor
   - Click on the block
   - Should NOT show JavaScript errors
   - Preview message should appear if no context

3. **Test cache invalidation:**
   - Edit entity data (paragraphs, media, etc.)
   - View page - should show updated data immediately
   - No stale cache issues

4. **Run code quality checks:**
   ```bash
   ddev exec vendor/bin/phpstan analyse web/modules/custom/{module}/
   ddev exec vendor/bin/phpcs web/modules/custom/{module}/
   ```

5. **Test multilingual:**
   - If site is multilingual, test in all languages
   - Verify translations display correctly

## Quick Reference

### Minimal Canvas-Compatible Block

```php
// 1. Simple annotation (no context_definitions)
@Block(
  id = "block_id",
  admin_label = @Translation("Label"),
  category = @Translation("Category")
)

// 2. Minimal dependencies (just route match)
public function __construct(
  array $configuration,
  $plugin_id,
  $plugin_definition,
  private readonly RouteMatchInterface $routeMatch,
) {
  parent::__construct($configuration, $plugin_id, $plugin_definition);
}

// 3. Default configuration
public function defaultConfiguration(): array {
  return ['label_display' => FALSE];
}

// 4. Empty block form
public function blockForm($form, FormStateInterface $form_state): array {
  $form = parent::blockForm($form, $form_state);
  return $form;
}

// 5. Preview message for Canvas
if (!$entity) {
  return ['#markup' => 'Preview message...'];
}

// 6. Proper cache tags
public function getCacheTags(): array {
  $tags = parent::getCacheTags();
  // Add entity + all referenced entities
  return $tags;
}
```

## Workflow Summary

1. **Create block plugin** with minimal dependencies
2. **Add theme hook** in module file
3. **Create template** using SDC components
4. **Add preview message** for Canvas
5. **Implement cache tags** for all referenced data
6. **Test** in both Canvas and live page
7. **Clear cache:** `ddev drush cr`
8. **Commit** with proper message

## Common Canvas Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| "Cannot read properties of undefined (reading 'source')" | Has `context_definitions` or no `blockForm()` | Remove context_definitions, add blockForm() |
| Block shows nothing in Canvas | Returns `[]` instead of preview | Add preview markup |
| "'{field}' is not a supported key" | Custom config field | Use entity fields instead of block config |
| Stale data after edits | Missing cache tags | Add tags for all referenced entities |
| "Property never read" PHPStan error | Unused injected service | Remove from constructor |
| NULL image validation error | Passing undefined keys to components | Use conditional rendering in template |

## Remember

- **Simplicity is key** - Don't over-engineer block configuration
- **Entity fields > Block config** - For dynamic data, use entity fields
- **Cache everything properly** - Include all referenced entities
- **Preview messages matter** - Make Canvas editor user-friendly
- **Test in both contexts** - Canvas editor AND live page
- **Follow Drupal standards** - PSR-4, dependency injection, coding standards

## Example: Real-World Implementation

See the MZBI Profession Facts Block implementation for a complete working example:
- `web/modules/custom/mzbi_blocks/src/Plugin/Block/ProfessionFactsBlock.php`
- Displays fact paragraphs from profession nodes
- Works in both Canvas and live pages
- Proper cache invalidation
- Clean, maintainable code
