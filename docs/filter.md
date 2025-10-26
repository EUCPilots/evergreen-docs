---
layout: doc
---
# Filter app details

## Filter with Where-Object

Where an application returns more than one object to the pipeline, you may want to filter the output with `Where-Object`. For example, `Get-EvergreenApp -Name MicrosoftTeams` returns both the `x86`, `x64`, and `ARM64` versions of the **Consumer** and **Enterprise** release rings ot the Microsoft Teams installer. To return just the 64-bit, Enterprise version of teams, we can filter the output with `Where-Object`:

```powershell
Get-EvergreenApp -Name "MicrosoftTeams" | Where-Object { $_.Architecture -eq "x64" -and $_.Release -eq "Enterprise" }
```

This will return details of the 64-bit Microsoft Teams installer that we can use in a script.

```powershell
Version      : 25275.2601.4002.2815
Release      : Enterprise
Architecture : x64
Type         : msix
URI          : https://installer.teams.static.microsoft/production-windows-x64/25275.2601.4002.2815/MSTeams-x64.msix
```

Or if we want the x64 and ARM64 version of Teams, we could change the `Where-Object` filter:

```powershell
Get-EvergreenApp -Name "MicrosoftTeams" | Where-Object { $_.Architecture -in "x64", "arm64" -and $_.Release -eq "Enterprise" }

Version      : 25275.2601.4002.2815
Release      : Enterprise
Architecture : arm64
Type         : msix
URI          : https://installer.teams.static.microsoft/production-windows-arm64/25275.2601.4002.2815/MSTeams-arm64.msix

Version      : 25275.2601.4002.2815
Release      : Enterprise
Architecture : x64
Type         : msix
URI          : https://installer.teams.static.microsoft/production-windows-x64/25275.2601.4002.2815/MSTeams-x64.msix
```

## Automatic Filtering

Evergreen supports automatic filtering of the output from `Get-EvergreenApp` and `Get-EvergreenAppFromApi` by adding output filters to the Evergreen apps cache directory. If the filter file exists, the output will always return the same filtered objects automatically.

The filters are added to the `/Evergreen/Filters` directory, along side the cached `Apps` and `Manifests` directories.. For example, adding `%LocalAppData%\Evergreen\Filters\MicrosoftTeams.json` with the following content will filter the output from `Get-EvergreenApp` in the same way as specifying `Where-Object` directly:

```json
{
    "filters": [
        {
            "property": "Release",
            "operator": "eq",
            "value": "Enterprise"
        },
        {
            "property": "Architecture",
            "operator": "in",
            "value": ["x64", "ARM64"]
        }
    ],
    "logicalOperator": "and"
}
```

This JSON is the same as specifying: `Where-Object { $_.Architecture -in "x64", "arm64" -and $_.Release -eq "Enterprise" }`.

::: tip
The filter file must be valid JSON and saved into the `/Evergreen/Filters` directory with the same name as the application function - e.g. `/Evergreen/Filters/MicrosoftTeams.json`.
:::

### Filter JSON Format

Filter configuration files allow you to define complex filtering logic for Evergreen application data. The JSON structure supports multiple filter conditions combined with logical operators (AND/OR).

### JSON Structure

#### Root Object

The root JSON object must contain the following properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `filters` | Array | Yes | An array of filter objects defining the conditions to apply |
| `logicalOperator` | String | No | How to combine multiple filters: `"and"` or `"or"` (defaults to `"and"`) |

#### Filter Object

Each filter object in the `filters` array must contain:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `property` | String | Yes | The name of the property to filter on (e.g., "Version", "Architecture", "Release") |
| `operator` | String | Yes | The comparison operator to use (see [Supported Operators](#supported-operators)) |
| `value` | String, Number, Array, or Boolean | Yes | The value to compare against (type depends on the operator) |

### Supported Operators

| Operator | Description | Value Type | Example |
|----------|-------------|------------|---------|
| `eq` | Equals | String, Number, Boolean | `"Enterprise"`, `1.0`, `true` |
| `ne` | Not equals | String, Number, Boolean | `"Professional"`, `2.0`, `false` |
| `like` | Wildcard match (PowerShell `-like`) | String | `"*Enterprise*"`, `"Win*"` |
| `match` | Regular expression match | String (regex pattern) | `"^2\\..*"`, `"[0-9]+"` |
| `in` | Value is in array | Array | `["x64", "ARM64"]`, `["Stable", "Beta"]` |
| `gt` | Greater than | Number, String | `30`, `"2.0.0"` |
| `lt` | Less than | Number, String | `100`, `"3.0.0"` |
| `ge` | Greater than or equal | Number, String | `18`, `"1.5.0"` |
| `le` | Less than or equal | Number, String | `65`, `"2.5.0"` |

**Note:** If an unknown operator is specified, `eq` (equals) is used as the default.

### Logical Operators

The `logicalOperator` property determines how multiple filters are combined:

- **`"and"`** (default): All filter conditions must be true for an item to match
- **`"or"`**: At least one filter condition must be true for an item to match

### Examples

#### Example 1: Simple AND Filter

Filter for Enterprise releases with x64 or ARM64 architecture:

```json
{
    "filters": [
        {
            "property": "Release",
            "operator": "eq",
            "value": "Enterprise"
        },
        {
            "property": "Architecture",
            "operator": "in",
            "value": ["x64", "ARM64"]
        }
    ],
    "logicalOperator": "and"
}
```

This matches items where:
- Release **equals** "Enterprise" **AND**
- Architecture is **in** the array ["x64", "ARM64"]

#### Example 2: OR Filter with Wildcards

Filter for items that are either Stable channel OR have "Enterprise" in the name:

```json
{
    "filters": [
        {
            "property": "Channel",
            "operator": "eq",
            "value": "Stable"
        },
        {
            "property": "Release",
            "operator": "like",
            "value": "*Enterprise*"
        }
    ],
    "logicalOperator": "or"
}
```

**Note:** `logicalOperator` is optional and defaults to `"and"` when omitted.

#### Example 3: Regular Expression Match

Filter for versions starting with "2." using regex:

```json
{
    "filters": [
        {
            "property": "Version",
            "operator": "match",
            "value": "^2\\."
        },
        {
            "property": "Channel",
            "operator": "eq",
            "value": "Stable"
        }
    ],
    "logicalOperator": "and"
}
```

**Note:** Remember to escape backslashes in JSON strings (`\\` instead of `\`).

#### Example 4: Exclude Specific Values

Filter to exclude x86 architecture:

```json
{
    "filters": [
        {
            "property": "Architecture",
            "operator": "ne",
            "value": "x86"
        }
    ]
}
```

#### Example 5: Complex Multi-Condition Filter

Filter for current stable Enterprise releases on specific architectures with version 2.0 or higher:

```json
{
    "filters": [
        {
            "property": "Release",
            "operator": "eq",
            "value": "Enterprise"
        },
        {
            "property": "Channel",
            "operator": "eq",
            "value": "Stable"
        },
        {
            "property": "Track",
            "operator": "eq",
            "value": "Current"
        },
        {
            "property": "Architecture",
            "operator": "in",
            "value": ["x64", "ARM64"]
        }
    ],
    "logicalOperator": "and"
}
```

### Best Practices

1. **Use descriptive file names**: Name your filter files clearly (e.g., `enterprise-x64-stable.json`)

2. **Use appropriate operators**:
   - Use `in` for multiple possible values
   - Use `like` for partial string matches
   - Use `match` for complex pattern matching
   - Use comparison operators (`gt`, `lt`, etc.) for numeric or version comparisons

3. **Property name accuracy**: Ensure property names exactly match the properties in your data (case-sensitive)

4. **Value types**: Match value types to your data:
   - Strings: `"value"`
   - Numbers: `123` (no quotes)
   - Arrays: `["value1", "value2"]`
   - Booleans: `true` or `false`

5. **Regular expressions**: Remember to escape special characters in JSON strings

6. **Logical operators**: Use `"or"` when any condition can match; use `"and"` (or omit) when all conditions must match

### Common Patterns

#### Filter by Multiple Architectures
```json
{
    "filters": [
        {
            "property": "Architecture",
            "operator": "in",
            "value": ["x64", "ARM64"]
        }
    ]
}
```

#### Filter Stable or Beta Channels
```json
{
    "filters": [
        {
            "property": "Channel",
            "operator": "in",
            "value": ["Stable", "Beta"]
        }
    ]
}
```

#### Exclude Preview Releases
```json
{
    "filters": [
        {
            "property": "Ring",
            "operator": "ne",
            "value": "Preview"
        }
    ]
}
```

### Troubleshooting

#### Filter Returns No Results

- Verify property names match exactly (including case)
- Check that values match the data type in your objects
- Test with verbose output: `Get-FilteredData -FilterPath "filter.json" -InputObject $data -Verbose`
- Simplify to one filter to isolate the issue

#### Filter Returns Too Many Results

- Verify `logicalOperator` is set correctly (`"and"` vs `"or"`)
- Check operator usage (ensure using `eq` not `like` if exact match needed)
- Add additional filter conditions to narrow results

#### JSON Parse Errors

- Validate JSON syntax using a JSON validator
- Ensure all strings are properly quoted
- Check for trailing commas (not allowed in JSON)
- Verify proper nesting of arrays and objects
