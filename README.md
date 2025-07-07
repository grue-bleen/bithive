# bithive

[![C++20](https://img.shields.io/badge/C%2B%2B-20%2F23%2F26-blue.svg)](https://en.cppreference.com/w/cpp/20)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A modern C++20/23/26 implementation of `std::hive` (also known as `std::colony`) optimized for `std::bitset` operations.

## Table of Contents

- [Introduction](#introduction)
- [Background: std::hive](#background-stdhive)
- [Why bithive?](#why-bithive)
- [Key Features](#key-features)
- [Usage Examples](#usage-examples)
- [Modern C++ Features](#modern-cpp-features)
- [Installation](#installation)
- [API Reference](#api-reference)
- [Requirements](#requirements)
- [Contributing](#contributing)
- [License](#license)

## Introduction

**bithive** is a specialized container that brings the benefits of `std::hive` (an unordered associative container with stable iterators) to the world of bitwise operations. It combines the memory efficiency and cache-friendly properties of `std::bitset` with the iterator stability and performance characteristics of `std::hive`.

Traditional bitwise containers often suffer from iterator invalidation during modifications, or require expensive memory reallocations. bithive solves these problems by providing a stable, efficient, and modern C++ interface for managing collections of bitsets.

## Background: std::hive

`std::hive` (formerly known as `std::colony`) is a proposed C++ container that provides:

- **Stable iterators**: Iterators remain valid even when elements are inserted or removed
- **Cache-friendly storage**: Elements are stored in contiguous memory blocks
- **Efficient insertion/deletion**: O(1) amortized insertion and deletion
- **Memory efficiency**: Minimal memory overhead compared to node-based containers

The container is particularly useful for scenarios where you need frequent insertions and deletions while maintaining iterator validity, such as game entities, particle systems, or any dynamic collection where references to elements must remain stable.

## Why bithive?

bithive extends the `std::hive` concept specifically for bitwise operations:

### Memory Efficiency
- **Compact storage**: Bitsets are stored in optimized memory layouts
- **Reduced fragmentation**: Intelligent memory block management
- **SIMD-friendly**: Aligned memory for vectorized operations

### Performance Benefits
- **Cache locality**: Related bitsets are stored close together in memory
- **Batch operations**: Efficient bulk bitwise operations across multiple bitsets
- **Iterator stability**: No iterator invalidation during container modifications

### Modern C++ Integration
- **Concepts**: Type-safe template interfaces
- **Ranges**: Full C++20 ranges support
- **Constexpr**: Compile-time evaluation where possible
- **CTAD**: Class Template Argument Deduction support

## Key Features

### 🚀 High Performance
- O(1) amortized insertion and deletion
- Cache-friendly memory layout
- SIMD-optimized bitwise operations
- Minimal memory overhead

### 🔒 Iterator Stability
- Iterators remain valid during insertions/deletions
- Reference stability for long-lived element access
- Safe concurrent read operations

### 🧬 Modern C++ Design
- Concepts-based template constraints
- Full ranges and algorithms support
- Constexpr evaluation support
- Exception safety guarantees

### 🎯 Bitset Optimization
- Specialized algorithms for bitwise operations
- Efficient bulk operations (AND, OR, XOR across collections)
- Custom allocator support for different bit layouts

## Usage Examples

### Basic Usage

```cpp
#include <bithive/bithive.hpp>
#include <iostream>

int main() {
    // Create a bithive for 64-bit bitsets
    bithive::hive<std::bitset<64>> bits;
    
    // Insert some bitsets
    auto it1 = bits.emplace(0b1010101010101010ULL);
    auto it2 = bits.emplace(0b1100110011001100ULL);
    auto it3 = bits.emplace(0b1111000011110000ULL);
    
    // Iterators remain stable even after modifications
    bits.emplace(0b0000111100001111ULL);
    
    // it1, it2, it3 are still valid!
    std::cout << "First bitset: " << *it1 << '\n';
    
    // Range-based iteration
    for (const auto& bitset : bits) {
        std::cout << "Bitset: " << bitset << '\n';
    }
    
    return 0;
}
```

### Advanced Operations with C++20 Ranges

```cpp
#include <bithive/bithive.hpp>
#include <ranges>
#include <algorithm>

// Using concepts for type safety
template<std::size_t N>
requires (N > 0 && N <= 1024)
void process_bitsets(bithive::hive<std::bitset<N>>& container) {
    namespace views = std::views;
    namespace ranges = std::ranges;
    
    // Find all bitsets with more than half bits set
    auto high_density = container 
        | views::filter([](const auto& bs) { 
            return bs.count() > N / 2; 
          });
    
    // Apply XOR with a mask to matching bitsets
    std::bitset<N> mask{0xAAAAAAAAULL};
    ranges::for_each(high_density, [&mask](auto& bs) {
        bs ^= mask;
    });
    
    // Bulk operations using parallel algorithms
    auto result_bitset = std::bitset<N>{};
    ranges::for_each(std::execution::par_unseq, container, 
        [&result_bitset](const auto& bs) {
            // Thread-safe bitwise OR (using atomic operations internally)
            result_bitset |= bs;
        });
}
```

### Custom Allocator and Memory Management

```cpp
#include <bithive/bithive.hpp>
#include <memory_resource>

// Using polymorphic memory resources
void memory_efficient_processing() {
    // Stack-based memory pool for small collections
    std::array<std::byte, 8192> buffer;
    std::pmr::monotonic_buffer_resource pool{buffer.data(), buffer.size()};
    
    // bithive with custom allocator
    bithive::hive<std::bitset<32>, std::pmr::polymorphic_allocator<std::bitset<32>>>
        bits{&pool};
    
    // Process data with zero heap allocations (if fits in buffer)
    for (std::uint32_t i = 0; i < 100; ++i) {
        bits.emplace(i * 0x12345678UL);
    }
    
    // Efficient bulk operations
    auto combined = bithive::algorithms::reduce_or(bits);
    std::cout << "Combined result: " << combined << '\n';
}
```

### Constexpr Operations (C++23/26)

```cpp
#include <bithive/bithive.hpp>

// Compile-time bitset processing
constexpr auto generate_lookup_table() {
    bithive::static_hive<std::bitset<8>, 256> lookup;
    
    // Generate all possible 8-bit patterns at compile-time
    for (std::size_t i = 0; i < 256; ++i) {
        lookup.emplace(static_cast<std::uint8_t>(i));
    }
    
    return lookup;
}

// Lookup table generated at compile time
constexpr auto bit_patterns = generate_lookup_table();
```

## Modern C++ Features

bithive leverages cutting-edge C++20/23/26 features:

### Concepts
```cpp
template<typename T>
concept BitsetLike = requires(T t) {
    typename T::reference;
    { t.size() } -> std::convertible_to<std::size_t>;
    { t.test(0) } -> std::convertible_to<bool>;
    { t.set(0) } -> std::same_as<T&>;
};

template<BitsetLike T>
class hive { /* ... */ };
```

### Ranges Support
```cpp
static_assert(std::ranges::range<bithive::hive<std::bitset<32>>>);
static_assert(std::ranges::view<bithive::hive<std::bitset<32>> | std::views::filter(predicate)>);
```

### Modules (C++20)
```cpp
import bithive;
import std.core;

// No header includes needed
```

### Coroutines Integration
```cpp
bithive::generator<std::bitset<64>> fibonacci_bitsets(std::size_t count) {
    std::uint64_t a = 0, b = 1;
    for (std::size_t i = 0; i < count; ++i) {
        co_yield std::bitset<64>{a};
        auto temp = a + b;
        a = b;
        b = temp;
    }
}
```

## Installation

### Requirements
- **Compiler**: GCC 11+, Clang 14+, or MSVC 19.30+ (Visual Studio 2022)
- **C++ Standard**: C++20 or later (C++23/26 recommended for full feature set)
- **CMake**: 3.20 or later

### Using CMake FetchContent

```cmake
include(FetchContent)

FetchContent_Declare(
    bithive
    GIT_REPOSITORY https://github.com/grue-bleen/bithive.git
    GIT_TAG        main
)

FetchContent_MakeAvailable(bithive)

target_link_libraries(your_target PRIVATE bithive::bithive)
```

### Manual Installation

```bash
git clone https://github.com/grue-bleen/bithive.git
cd bithive
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
cmake --install .
```

### Package Managers

#### Conan
```bash
conan install bithive/1.0.0@
```

#### vcpkg
```bash
vcpkg install bithive
```

## API Reference

### Core Container

#### `bithive::hive<T, Allocator>`

```cpp
template<BitsetLike T, typename Allocator = std::allocator<T>>
class hive {
public:
    // Type definitions
    using value_type = T;
    using allocator_type = Allocator;
    using size_type = std::size_t;
    using iterator = /* implementation-defined */;
    using const_iterator = /* implementation-defined */;
    
    // Constructors
    hive() = default;
    explicit hive(const Allocator& alloc);
    
    // Element access
    iterator emplace(auto&&... args);
    iterator insert(const T& value);
    iterator insert(T&& value);
    
    // Modifiers
    void erase(const_iterator it);
    void erase(const_iterator first, const_iterator last);
    void clear() noexcept;
    
    // Capacity
    [[nodiscard]] bool empty() const noexcept;
    [[nodiscard]] size_type size() const noexcept;
    [[nodiscard]] size_type capacity() const noexcept;
    
    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
};
```

### Algorithms

#### Bitwise Operations
```cpp
namespace bithive::algorithms {
    // Reduce operations
    template<BitsetLike T>
    constexpr T reduce_or(const hive<T>& container);
    
    template<BitsetLike T>
    constexpr T reduce_and(const hive<T>& container);
    
    template<BitsetLike T>
    constexpr T reduce_xor(const hive<T>& container);
    
    // Parallel operations
    template<std::execution::execution_policy ExecPolicy, BitsetLike T>
    constexpr T parallel_reduce_or(ExecPolicy&& policy, const hive<T>& container);
}
```

## Requirements

### Compiler Support

| Compiler | Minimum Version | C++20 | C++23 | C++26 |
|----------|----------------|-------|-------|-------|
| GCC      | 11.0           | ✅    | ✅    | 🚧    |
| Clang    | 14.0           | ✅    | ✅    | 🚧    |
| MSVC     | 19.30          | ✅    | ✅    | 🚧    |

### Feature Matrix

| Feature | C++20 | C++23 | C++26 | Notes |
|---------|-------|-------|-------|-------|
| Basic Container | ✅ | ✅ | ✅ | Full support |
| Concepts | ✅ | ✅ | ✅ | Type safety |
| Ranges | ✅ | ✅ | ✅ | std::ranges integration |
| Constexpr Operations | ⚠️ | ✅ | ✅ | Limited in C++20 |
| Modules | ✅ | ✅ | ✅ | Optional |
| Coroutines | ✅ | ✅ | ✅ | Generator support |
| Static Reflection | ❌ | ❌ | 🚧 | Future feature |

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Setup

```bash
git clone https://github.com/grue-bleen/bithive.git
cd bithive
cmake -B build -DBITHIVE_BUILD_TESTS=ON -DBITHIVE_BUILD_EXAMPLES=ON
cmake --build build
ctest --test-dir build
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
