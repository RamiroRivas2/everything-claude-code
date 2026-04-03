---
description: Generate Dart Freezed models from Supabase table schemas
---

# Generate Dart Types from Supabase Schema

Inspect Supabase tables and generate matching Dart Freezed models.

## Process

1. Read the table schema (via MCP if connected, or from migration files in `supabase/migrations/`)
2. Map SQL types to Dart types:
   - `uuid` -> `String`
   - `text` / `varchar` -> `String`
   - `int4` / `int8` -> `int`
   - `float4` / `float8` -> `double`
   - `bool` -> `bool`
   - `timestamptz` -> `DateTime`
   - `jsonb` -> `Map<String, dynamic>` (or a typed Freezed class if structure is known)
   - `uuid[]` / `text[]` -> `List<String>`
   - Custom enums -> Dart `enum` with `@JsonEnum`
3. Handle nullability: `not null` -> required, nullable -> `Type?`
4. Handle defaults: server-generated columns (`created_at`, `updated_at`) should be nullable and excluded from insert payloads — `@Default` only works with compile-time constants
5. Map `snake_case` SQL columns to `camelCase` Dart fields with `@JsonKey(name:)`

## Output Per Table

```dart
@freezed
class TableName with _$TableName {
  const factory TableName({
    required String id,
    @JsonKey(name: 'user_id') required String userId,
    required String title,
    String? description,
    @JsonKey(name: 'is_active') @Default(false) bool isActive,
    @JsonKey(name: 'created_at') DateTime? createdAt,
    @JsonKey(name: 'updated_at') DateTime? updatedAt,
  }) = _TableName;

  factory TableName.fromJson(Map<String, dynamic> json) =>
      _$TableNameFromJson(json);
}

extension TableNameSupabase on TableName {
  Map<String, dynamic> toInsertJson() => toJson()
    ..remove('id')
    ..remove('created_at')
    ..remove('updated_at');
}
```

## Output

1. One Freezed model file per table
2. Barrel export file if multiple models generated
3. Remind: `dart run build_runner build --delete-conflicting-outputs`
