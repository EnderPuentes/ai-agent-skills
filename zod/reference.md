# Zod — Reference & Official Documentation

This file complements the zod skill with official documentation links and API sections for indexing (aligned with Zod’s public docs and llms.txt).

## Official documentation (indexable)

- **Home**: https://zod.dev
- **Introduction / basics**: https://zod.dev/basics (inferring types, input vs output)
- **API (Zod 4)**: https://zod.dev/api — Full reference for schema types and methods
- **LLMs / agents**: https://zod.dev/llms.txt — Structured documentation for tools and indexation

## API sections (Zod 4 — from llms.txt / docs)

Use these for deep lookup; base URL is https://zod.dev/api unless noted.

- **Primitives**: Primitives
- **Coercion**: Coercion
- **Literals**: Literals
- **Strings**: Strings
- **String formats**: String formats, Emails, UUIDs, URLs, ISO datetimes, ISO dates, ISO times, IP addresses, IP blocks (CIDR), MAC Addresses, JWTs, Hashes, Custom formats, Template literals
- **Numbers**: Numbers, Integers, BigInts
- **Booleans**: Booleans
- **Dates**: Dates
- **Enums**: Enums, .enum, .exclude(), .extract()
- **Stringbools**: Stringbools
- **Optionals / Nullables / Nullish**: Optionals, Nullables, Nullish
- **Unknown / Never**: Unknown, Never
- **Objects**: Objects, z.strictObject, z.looseObject, .catchall(), .shape, .keyof(), .extend(), .safeExtend(), .pick(), .omit(), .partial(), .required(), Recursive objects, Circularity errors
- **Arrays**: Arrays
- **Tuples**: Tuples
- **Unions**: Unions, Exclusive unions (XOR), Discriminated unions
- **Intersections**: Intersections
- **Records**: Records, z.record, z.partialRecord, z.looseRecord
- **Maps / Sets**: Maps, Sets
- **Files**: Files
- **Promises**: Promises
- **Instanceof**: Instanceof
- **Property**: Property
- **Refinements**: Refinements, .refine(), error
- **Transform / pipe**: Transform, pipe (see API)
- **Error handling**: Error, ZodError, .format()

## Zod Mini (tree-shakable)

- **Zod Mini**: https://zod.dev/packages/mini — Tree-shaking, when (not) to use, DX, backend, ZodMiniType, .parse, .check(), .register(), .brand(), .clone(def), no default locale

## Ecosystem

- **Ecosystem**: https://zod.dev (Ecosystem section) — Form integrations (e.g. React Hook Form with zodResolver), API libraries, Zod to X, X to Zod, mocking
- **React Hook Form**: Use `@hookform/resolvers` with `zodResolver(schema)`; type form with `z.infer<typeof schema>`
- **tRPC**: End-to-end typesafe APIs with Zod schemas

## Requirements (reminder)

- TypeScript 5.5+ recommended
- **strict** mode enabled in `tsconfig.json`:
  ```json
  { "compilerOptions": { "strict": true } }
  ```

## Installation

```bash
npm install zod
```

Also available as `@zod/zod` on jsr.io. Zod provides an MCP server for agent/editor integration; see https://zod.dev for instructions.
