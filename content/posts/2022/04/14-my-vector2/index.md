---
title: "c++98で自作 vector (2)"
date: 2022-04-14T21:48:22+09:00
description: 自作vector実装編
draft: true
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-my-vector2"
---

## 1. はじめに
* * *
[前回の基礎知識編](https://public-jun.github.io/cpp-my-vector1/)の続き、実装編です。
この記事では、挿入(insert)できるところまで作ります。

## 2. 概要を掴む 
* * *
実装のイメージをつけるために、[江添亮のC++入門](https://cpp.rainy.me/)の[std::array](https://cpp.rainy.me/020-array.html)から [vectorの実装 : メモリー確保](https://cpp.rainy.me/034-vector-memory-allocation.html)までを写経しました。

この資料ではイテレータをポインタのエイリアスで実装しています。

[llvm](https://github.com/llvm/llvm-project/blob/main/libcxx/include/vector) や [gcc](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/stl_vector.h) はイテレータを用意しているので、最終的にはポインタで操作しているところをイテレータに置き換えていきます。

## 3. 実装
自作 vector は `ft`という名前空間に作ります。
`ft::vector` で呼び出します。
### イテレータ

**iterator_traits.hpp**
```cpp
#ifndef ITERATOR_TRAITS_HPP
#define ITERATOR_TRAITS_HPP

#include <cstddef> // ptrdiff_t
#include <iterator>

namespace ft {

/// primary iterator_traits
template <class Iterator>
struct iterator_traits {
    typedef typename Iterator::difference_type   difference_type;
    typedef typename Iterator::value_type        value_type;
    typedef typename Iterator::pointer           pointer;
    typedef typename Iterator::reference         reference;
    typedef typename Iterator::iterator_category iterator_category;
};

/// Partial specialization for pointer types
template <class T>
struct iterator_traits<T*> {
    typedef ptrdiff_t                       difference_type;
    typedef T                               value_type;
    typedef T*                              pointer;
    typedef T&                              reference;
    typedef std::random_access_iterator_tag iterator_category;
};

/// Partial specialization for const pointer types.
template <class T>
struct iterator_traits<const T*> {
    typedef ptrdiff_t                       difference_type;
    typedef T                               value_type;
    typedef const T*                        pointer;
    typedef const T&                        reference;
    typedef std::random_access_iterator_tag iterator_category;
};
} // namespace ft

#endif
```

### vector insert 
```cpp
template <typename T, typename Allocator = std::allocator<T> >
class vector
{
public:
    // Member type
    typedef T                                    value_type;
    typedef Allocator                            allocator_type;
    typedef std::size_t                          size_type;
    typedef std::ptrdiff_t                       difference_type;
    typedef value_type&                          reference;
    typedef const value_type&                    const_reference;
    typedef typename Allocator::pointer          pointer;
    typedef typename Allocator::const_pointer    const_pointer;
    typedef wrap_iter<pointer>                   iterator;
    typedef wrap_iter<const_pointer>             const_iterator;
    typedef ft::reverse_iterator<iterator>       reverse_iterator;
    typedef ft::reverse_iterator<const_iterator> const_reverse_iterator;

    // Member functions
    vector()
        : first_(NULL), last_(NULL), capacity_last_(NULL),
          alloc_(allocator_type())
    {}

    explicit vector(const allocator_type& alloc)
        : first_(NULL), last_(NULL), capacity_last_(NULL), alloc_(alloc)
    {}

    vector(size_type count, const T& value = T(),
           const allocator_type& alloc = allocator_type())
        : first_(NULL), last_(NULL), capacity_last_(NULL), alloc_(alloc)
    {
        resize(count, value);
    }

    template <typename InputIterator>
    vector(InputIterator first, InputIterator last,
           const Allocator& allocator                   = Allocator(),
           typename ft::enable_if<!ft::is_integral<InputIterator>::value,
                                  InputIterator>::type* = NULL)
        : first_(NULL), last_(NULL), capacity_last_(NULL), alloc_(allocator)
    {
        __range_initialize(
            first, last,
            typename iterator_traits<InputIterator>::iterator_category());
    }

    vector(const vector& r)
        : first_(NULL), last_(NULL), capacity_last_(NULL), alloc_(r.alloc_)
    {
        reserve(r.size());
        pointer dest = first_;
        for (const_iterator src = r.begin(), last = r.end(); src != last;
             ++dest, ++src)
        {
            __construct(dest, *src);
        }
        last_ = first_ + r.size();
    }

    vector& operator=(const vector& r)
    {
        if (this == &r)
            return *this;
        if (size() == r.size())
        {
            std::copy(r.begin(), r.end(), begin());
        }
        else if (capacity() >= r.size())
        {
            std::copy(r.begin(), r.begin() + r.size(), begin());
            last_ = first_ + r.size();
        }
        else
        {
            __destroy_until(rend()); // destroy_all()
            reserve(r.size());
            pointer dest_pointer = first_;
            for (const_iterator src_iter = r.begin(), src_end = r.end();
                 src_iter != src_end; ++src_iter, ++dest_pointer, ++last_)
            {
                __construct(dest_pointer, *src_iter);
            }
        }
        return *this;
    }

    ~vector()
    {
        clear();
        __deallocate();
    }

    void assign(size_type n, const T& value)
    {
        if (capacity() >= n)
        {
            size_type sz = size();
            if (size() > n)
                __destroy_until(rbegin() + (sz - n));
            std::fill_n(first_, n, value);
        }
        else
        {
            clear();
            __deallocate();
            __allocate(n);
            std::uninitialized_fill_n(first_, n, value);
        }
        last_ = first_ + n;
    }

    template <class InputIterator>
    void assign(InputIterator first, InputIterator last,
                typename ft::enable_if<!ft::is_integral<InputIterator>::value,
                                       InputIterator>::type* = NULL)
    {
        size_type new_size = static_cast<size_type>(std::distance(first, last));
        if (new_size <= capacity())
        {
            InputIterator mid = last;
            if (new_size > size())
            {
                mid = first;
                std::advance(mid, size());
                std::copy(first, mid, begin());
                for (InputIterator it = mid; it != last; ++it)
                    __construct(last_++, *it);
            }
            else
            {
                std::copy(first, mid, begin());
                difference_type diff = size() - new_size;
                __destroy_until(rbegin() + diff);
            }
        }
        else
        {
            clear();
            __deallocate();
            __allocate(new_size);
            for (InputIterator it = first; it != last; ++it)
                __construct(last_++, *it);
        }
    }

    allocator_type get_allocator() const { return (allocator_type(alloc_)); }

    size_type max_size() const
    {
        return std::min<size_type>(alloc_.max_size(),
                                   std::numeric_limits<difference_type>::max());
    }

    size_type size() const { return end() - begin(); }
    bool      empty() const { return begin() == end(); }
    size_type capacity() const { return capacity_last_ - first_; }

    reference       operator[](size_type i) { return first_[i]; }
    const_reference operator[](size_type i) const { return first_[i]; }
    reference       at(size_type i)
    {
        if (i >= size())
            __throw_out_of_range();
        return first_[i];
    }
    const_reference at(size_type i) const
    {
        if (i >= size())
            __throw_out_of_range();
        return first_[i];
    }
    reference       front() { return *first_; }
    const_reference front() const { return *first_; }
    reference       back() { return *(last_ - 1); }
    const_reference back() const { return *(last_ - 1); }

    iterator               begin() { return iterator(first_); }
    iterator               end() { return iterator(last_); }
    const_iterator         begin() const { return const_iterator(first_); }
    const_iterator         end() const { return const_iterator(last_); }
    reverse_iterator       rbegin() { return reverse_iterator(end()); }
    reverse_iterator       rend() { return reverse_iterator(begin()); }
    const_reverse_iterator rbegin() const
    {
        return const_reverse_iterator(begin());
    }
    const_reverse_iterator rend() const
    {
        return const_reverse_iterator(end());
    }

    void clear() { __destroy_until(rend()); }

    iterator insert(iterator pos, const_reference value)
    {
        difference_type diff = pos - begin();
        insert(pos, 1, value);
        pointer p_pos = first_ + diff;
        return iterator(p_pos);
    }

    void insert(iterator pos, size_type count, const_reference value)
    {
        difference_type offset   = pos - begin();
        size_type       new_size = size() + count;

        if (count > 0)
        {
            if (new_size >= capacity())
            {
                __extend_capacity(count);
            }

            pointer p_pos    = first_ + offset;
            pointer old_last = last_;

            size_type after_pos_size = static_cast<size_type>(last_ - p_pos);
            size_type left_count     = count;

            __move_range(p_pos, old_last, count);
            if (count > after_pos_size)
            {
                size_type unini_size = count - after_pos_size;
                std::uninitialized_fill_n(old_last, unini_size, value);
                left_count -= unini_size;
            }
            if (left_count > 0)
            {
                std::fill_n(p_pos, left_count, value);
            }
            last_ += count;
        }
    }

    template <class InputIterator>
    typename ft::enable_if<!ft::is_integral<InputIterator>::value,
                           void>::type // void
    insert(iterator pos, InputIterator first, InputIterator last)
    {
        __range_insert(
            pos, first, last,
            typename iterator_traits<InputIterator>::iterator_category());
    }

    iterator erase(iterator pos)
    {
        if (pos != end())
        {
            std::copy(pos + 1, end(), pos);
            --last_;
            __destroy(last_);
        }
        return (pos);
    }

    iterator erase(iterator first, iterator last)
    {
        difference_type erase_size = std::distance(first, last);
        if (first != last)
        {
            if (last != end())
                std::copy(last, end(), first);
            __destroy_until(rbegin() + erase_size);
        }
        return first;
    }

    void reserve(size_type sz)
    {
        if (sz <= capacity())
            return;
        if (sz > max_size())
            __throw_length_error();
        pointer   old_first    = first_;
        pointer   old_last     = last_;
        size_type old_capacity = capacity();
        __allocate(sz);
        for (pointer old_iter = old_first; old_iter != old_last;
             ++old_iter, ++last_)
        {
            __construct(last_, *old_iter);
        }
        for (pointer riter = old_first; riter != old_last; ++riter)
        {
            __destroy(riter);
        }
        alloc_.deallocate(old_first, old_capacity);
    }

    void push_back(const_reference value)
    {
        __extend_capacity(1);
        __construct(last_, value);
        ++last_;
    }

    void pop_back() { __destroy_until(rbegin() + 1); }

    void resize(size_type count, value_type value = value_type())
    {
        if (count < size())
        {
            erase(end() - (size() - count), end());
        }
        else if (count > size())
        {
            insert(end(), count - size(), value);
        }
    }

    void swap(vector& other)
    {
        std::swap(first_, other.first_);
        std::swap(last_, other.last_);
        std::swap(capacity_last_, other.capacity_last_);
    }

private:
    pointer        first_;
    pointer        last_;
    pointer        capacity_last_;
    allocator_type alloc_;

//
// private functions ....
//
}
```
### x. Markdown CheetSheet

#### Text Format

_Italic（斜体）_
*Italic（斜体）*

__Emphasis（強調）__
**Emphasis（強調）**

~~Strikethrough（取り消し線）~~

<details><summary>これは詳細表示の例です。</summary>詳細をこっちに書きます。</details>

This is `inline`.

### List
* text
    * test
    * test

- text
    - test
    - test

1. text
1. test
    1. test

#### Horizontal rules
* * *
***
*****
- - -
---------------------------------------

#### Blockquotes（引用）
> This is Blockquotes

#### Links（参照）
[]()

#### Images（画像）
![]()

#### Tables（表）
| id     | name    | date       |
| ------ | ------- | ---------- |
| 1      | test    | 2019-01-01 |
| 2      | test    | 2019-01-02 |
| 3      | test    | 2019-01-03 |
