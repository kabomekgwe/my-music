# Naming Conventions

**Type**: Always  
**Description**: Mandatory file and variable naming standards for consistency across the codebase

## File Naming (MANDATORY)

- **Components**: `PascalCase.tsx` → `UserProfileCard.tsx`
- **Pages**: `kebab-case/page.tsx` → `user-profile/page.tsx`
- **API Routes**: `kebab-case/route.ts` → `user-profiles/route.ts`
- **Types**: `kebab-case.types.ts` → `user-profile.types.ts`
- **Services**: `kebab-case.service.ts` → `user-profile.service.ts`
- **Hooks**: `use-kebab-case.ts` → `use-user-profile.ts`

## Variable Naming (MANDATORY)

- **Booleans**: `is/has/can/should + PascalCase` → `isUserActive`, `hasPermission`
- **Arrays**: `plural noun` → `users`, `permissions`, `activeUsers`
- **Functions**: `verb + noun` → `getUserById`, `validateUserInput`
- **Constants**: `SCREAMING_SNAKE_CASE` → `MAX_RETRY_ATTEMPTS`
- **Event Handlers**: `handle + Action` → `handleSubmit`, `handleUserClick`

## API Naming (MANDATORY)

- **Endpoints**: `kebab-case` → `/api/v1/user-profiles/{id}`
- **Query Params**: `camelCase` → `?sortBy=createdAt&isActive=true`
- **JSON Fields**: `camelCase` → `{ "firstName": "John", "isActive": true }`

## Examples

### ✅ Correct Naming

```typescript
// File: user-profile-card.tsx
interface UserProfileCardProps {
  user: User;
  isEditable: boolean;
  onUserUpdate: (user: User) => void;
}

export function UserProfileCard({ user, isEditable, onUserUpdate }: UserProfileCardProps) {
  const [isLoading, setIsLoading] = useState(false);
  const activeUsers = users.filter(user => user.isActive);
  
  const handleSubmit = async (formData: FormData) => {
    // Handle form submission
  };
  
  return (
    <div className="user-profile-card">
      {/* Component content */}
    </div>
  );
}
```

```typescript
// File: user-profile.service.ts
@Injectable()
export class UserProfileService {
  async getUserById(id: string): Promise<UserResponseDto> {
    // Service implementation
  }
  
  async validateUserInput(input: CreateUserDto): Promise<boolean> {
    // Validation logic
  }
}
```

### ❌ Incorrect Naming

```typescript
// File: UserProfileCard.tsx (wrong - should be kebab-case for pages/routes)
// File: user_profile_card.tsx (wrong - should use kebab-case, not snake_case)

interface Props { // wrong - should be specific: UserProfileCardProps
  data: any; // wrong - should be typed: user: User
  loading: boolean; // wrong - should be: isLoading: boolean
}

export function UserProfileCard({ data, loading }: Props) {
  const [Loading, setLoading] = useState(false); // wrong - should be: isLoading
  const Users = users.filter(user => user.active); // wrong - should be: activeUsers
  
  const Submit = async (formData: FormData) => { // wrong - should be: handleSubmit
    // Handle form submission
  };
}
```

## Quick Reference

| Type | Pattern | Example |
|------|---------|---------|
| React Components | `PascalCase.tsx` | `UserProfileCard.tsx` |
| Pages/Routes | `kebab-case/page.tsx` | `user-profile/page.tsx` |
| Services | `kebab-case.service.ts` | `user-profile.service.ts` |
| Types | `kebab-case.types.ts` | `user-profile.types.ts` |
| Hooks | `use-kebab-case.ts` | `use-user-profile.ts` |
| Booleans | `is/has/can/should + PascalCase` | `isUserActive` |
| Arrays | `plural noun` | `users`, `activeUsers` |
| Functions | `verb + noun` | `getUserById` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_ATTEMPTS` |
| Event Handlers | `handle + Action` | `handleSubmit` |
| API Endpoints | `kebab-case` | `/api/v1/user-profiles` |
| Query Params | `camelCase` | `?sortBy=createdAt` |
| JSON Fields | `camelCase` | `{ "firstName": "John" }` |
