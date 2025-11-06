# Contributing to Android Interview Preparation Guide

Thank you for your interest in contributing to this project! This document provides guidelines and instructions for contributing.

## 🎯 How to Contribute

We welcome contributions in the following areas:

### 1. Content Improvements
- Fix typos and grammatical errors
- Improve clarity of explanations
- Add more code examples
- Enhance question quality
- Update outdated information

### 2. New Content
- Add new topics
- Provide more interview questions
- Share real-world examples
- Document best practices
- Create additional resources

### 3. Code Examples
- Improve existing code examples
- Add error handling
- Provide alternative solutions
- Add performance optimizations
- Include unit tests

### 4. Translations
- Translate content to other languages
- Maintain consistency across translations

## 📝 Contribution Process

### Step 1: Fork the Repository
```bash
# Fork the repository on GitHub
git clone https://github.com/mukundjogi/android-interview.git
cd android-interview
```

### Step 2: Create a Branch
```bash
# Create a feature branch
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

### Step 3: Make Changes
- Make your changes following the guidelines below
- Test your changes locally
- Ensure all content is accurate and up-to-date

### Step 4: Commit Your Changes
```bash
git add .
git commit -m "Add: Brief description of changes"
```

**Commit Message Guidelines:**
- Use prefixes: `Add:`, `Fix:`, `Update:`, `Remove:`
- Keep messages concise and descriptive
- Example: `Add: Memory leak examples in memory-management.md`

### Step 5: Push and Create Pull Request
```bash
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub.

## ✍️ Content Guidelines

### Code Examples

1. **Use Kotlin** - All code should be in Kotlin
2. **Production-Ready** - Code should compile and work
3. **Commented** - Add comments for complex logic
4. **Best Practices** - Follow Android best practices
5. **Formatting** - Use proper indentation and spacing

**Good Example:**
```kotlin
/**
 * ViewModel for managing user authentication state
 * Uses StateFlow for state management
 */
class AuthViewModel(
    private val authRepository: AuthRepository
) : ViewModel() {
    
    private val _authState = MutableStateFlow<AuthState>(AuthState.Idle)
    val authState: StateFlow<AuthState> = _authState.asStateFlow()
    
    fun signIn(email: String, password: String) {
        viewModelScope.launch {
            _authState.value = AuthState.Loading
            try {
                val user = authRepository.signIn(email, password)
                _authState.value = AuthState.Success(user)
            } catch (e: Exception) {
                _authState.value = AuthState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

### Question Format

When adding questions, use this structure:

```markdown
### Q: Your question here?

**Answer:**

Explanation of the answer...

```kotlin
// Code example if applicable
class Example { }
```

**Key Points:**
- Point 1
- Point 2
- Point 3
```

### Document Structure

Each topic file should include:

1. **Front Matter** (at the top)
2. **Title** (H1)
3. **Table of Contents** with anchor links
4. **Introduction** section
5. **Main Content** with subsections
6. **Code Examples** throughout
7. **Interview Questions & Answers** section
8. **Navigation** links (Previous/Next)
9. **References** (optional)

### Writing Style

- **Clear and Concise**: Write clearly and directly
- **Technical Accuracy**: Ensure all technical information is correct
- **Beginner-Friendly**: Explain concepts clearly
- **Code-Heavy**: Include practical code examples
- **Up-to-Date**: Keep content current with latest Android versions

## 🔍 Review Process

### What We Look For

1. **Accuracy**: Information must be technically correct
2. **Clarity**: Content should be easy to understand
3. **Completeness**: Address the topic thoroughly
4. **Code Quality**: Code should follow best practices
5. **Consistency**: Match existing style and format

### Types of Review Feedback

We may ask for:
- Clarifications on technical points
- Additional code examples
- Better explanations
- Formatting improvements
- Testing of code examples

## 📚 Areas That Need Contribution

### High Priority
- [ ] More Kotlin examples in all topics
- [ ] Additional interview questions
- [ ] Architecture pattern comparisons
- [ ] Performance optimization examples
- [ ] Testing strategies for each topic

### Medium Priority
- [ ] Add diagrams and illustrations
- [ ] Video tutorials/explanations
- [ ] Interview simulation scenarios
- [ ] Company-specific interview prep
- [ ] Mock interview templates

### Low Priority
- [ ] Translations to other languages
- [ ] Interview success stories
- [ ] Career advice sections
- [ ] Networking opportunities
- [ ] Resource links

## 🐛 Reporting Issues

### Bug Reports

If you find a bug or error:

1. Check if issue already exists
2. Provide clear description
3. Include steps to reproduce
4. Add screenshots if applicable
5. Specify the file and section

**Bug Report Template:**
```markdown
**File:** `docs/android-basics.md`
**Section:** Activity Lifecycle
**Issue:** Incorrect explanation of onResume()
**Description:** The documentation states...
**Expected:** Should state...
**Actual:** Currently states...
```

### Feature Requests

For new features:
- Clearly describe the feature
- Explain why it's needed
- Suggest implementation approach

## 💡 Tips for Contributors

### Before Contributing

1. Read through existing content to understand style
2. Check open issues and pull requests
3. Test any code examples you add
4. Run the site locally to preview changes
5. Ensure content is accurate and up-to-date

### Getting Help

- Check existing documentation
- Ask questions in issues
- Join discussions
- Reach out to maintainers

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

## 🙏 Recognition

Contributors will be:
- Credited in the README
- Listed in CONTRIBUTORS.md (to be created)
- Acknowledged in release notes

## 🎉 Thank You!

Your contributions make this project better for everyone. We appreciate your time and effort in making Android interview preparation accessible to all!

---

**Questions?** Open an issue or start a discussion!

