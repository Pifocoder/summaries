



```C++
#include <iostream>  
#include <vector>  
  
template <typename Type, typename Alloc = std::allocator<Type>>  
class List {  
private:  
struct BaseNode {  
BaseNode* next = nullptr;  
};  
struct Node : public BaseNode {  
Type data;  
  
// using type_traits =  
// typename std::allocator_traits<Alloc>::template  
// rebind_traits<Node>; using type_alloc = typename  
// std::allocator_traits<Alloc>::template rebind_alloc<Node>;  
  
Node() : data(Type()) {}  
  
template <typename... Args>  
Node(Args&&... args) : data(std::forward<Args>(args)...) {}  
~Node() = default;  
};  
  
using node_traits =  
typename std::allocator_traits<Alloc>::template rebind_traits<Node>;  
using node_alloc =  
typename std::allocator_traits<Alloc>::template rebind_alloc<Node>;  
  
[[no_unique_address]] node_alloc alloc_node_;  
BaseNode* head_ = nullptr;  
size_t size_ = 0;  
  
public:  
void update_head();  
List() { update_head(); }  
template <typename... Args>  
List(const size_t& size, const Args&... value);  
List(size_t size);  
List(size_t size, const Alloc& alloc);  
List(const Alloc& alloc)  
: alloc_node_(node_traits::select_on_container_copy_construction(alloc)) {  
update_head();  
}  
template <typename... Args>  
List(size_t size, const Args&... value, const Alloc& alloc)  
: alloc_node_(node_traits::select_on_container_copy_construction(alloc)) {  
(*this) = List(size, value...);  
}  
List(List&& other) : head_(other.head_), size_(other.size_) {  
other.head_ = nullptr;  
other.size_ = 0;  
}  
List(const List& other);  
  
const node_alloc& get_allocator() const { return (alloc_node_); }  
size_t size() const { return size_; }  
  
void move_after(BaseNode* position, BaseNode* data) {  
auto last_next = position->next;  
position->next = data;  
data->next = last_next;  
}  
  
List& operator=(List&& other) noexcept {  
List copy = std::move(other);  
std::swap(head_, copy.head_);  
std::swap(size_, copy.size_);  
return *this;  
}  
List& operator=(const List& other);  
  
template <bool IsConst>  
class CommonIterator;  
  
using iterator = CommonIterator<false>;  
using const_iterator = CommonIterator<true>;  
using reverse_iterator = std::reverse_iterator<iterator>;  
using const_reverse_iterator = std::reverse_iterator<const_iterator>;  
  
List<Type, Alloc>::iterator begin();  
List<Type, Alloc>::iterator end();  
  
List<Type, Alloc>::const_iterator begin() const;  
List<Type, Alloc>::const_iterator end() const;  
  
List<Type, Alloc>::const_iterator cbegin() const;  
List<Type, Alloc>::const_iterator cend() const;  
  
List<Type, Alloc>::reverse_iterator rbegin();  
List<Type, Alloc>::reverse_iterator rend();  
  
List<Type, Alloc>::const_reverse_iterator rbegin() const;  
List<Type, Alloc>::const_reverse_iterator rend() const;  
  
List<Type, Alloc>::const_reverse_iterator crbegin() const;  
List<Type, Alloc>::const_reverse_iterator crend() const;  
  
template <typename... Args>  
void insert_after(List<Type, Alloc>::const_iterator iter, Args&&... value);  
void erase_after(List<Type, Alloc>::const_iterator iter);  
  
~List();  
};  
template <typename Type, typename Alloc>  
void List<Type, Alloc>::update_head() {  
if (head_ == nullptr) {  
head_ = node_traits::allocate(alloc_node_, 1);  
head_->next = head_;  
}  
}  
template <typename Type, typename Alloc>  
template <typename... Args>  
List<Type, Alloc>::List(const size_t& size, const Args&... value)  
: size_(size) {  
BaseNode* last;  
update_head();  
for (size_t i = 0; i < size; ++i) {  
Node* data = node_traits::allocate(alloc_node_, 1);  
BaseNode* object = static_cast<BaseNode*>(data);  
try {  
node_traits::construct(alloc_node_, data, value...);  
} catch (...) {  
node_traits::destroy(alloc_node_, data);  
}  
  
if (i > 0) {  
last->next = object;  
std::swap(last, object);  
} else {  
last = object;  
head_->next = object;  
}  
if (i == size - 1) {  
last->next = head_;  
}  
}  
}  
  
template <typename Type, typename Alloc>  
List<Type, Alloc>::List(size_t size) : size_(size) {  
BaseNode* last;  
update_head();  
for (size_t i = 0; i < size; ++i) {  
Node* data = node_traits::allocate(alloc_node_, 1);  
try {  
node_traits::construct(alloc_node_, data);  
} catch (...) {  
node_traits::destroy(alloc_node_, data);  
}  
BaseNode* object = static_cast<BaseNode*>(data);  
if (i > 0) {  
last->next = object;  
std::swap(last, object);  
} else {  
last = object;  
head_->next = object;  
}  
if (i == size - 1) {  
last->next = head_;  
}  
}  
}  
template <typename Type, typename Alloc>  
List<Type, Alloc>::List(size_t size, const Alloc& alloc)  
: alloc_node_(alloc), size_(size) {  
BaseNode* last;  
update_head();  
for (size_t i = 0; i < size; ++i) {  
Node* data = node_traits::allocate(alloc_node_, 1);  
try {  
node_traits::construct(alloc_node_, data);  
} catch (...) {  
node_traits::destroy(alloc_node_, data);  
}  
  
BaseNode* object = static_cast<BaseNode*>(data);  
  
if (i > 0) {  
last->next = object;  
std::swap(last, object);  
} else {  
last = object;  
head_->next = object;  
}  
if (i == size - 1) {  
last->next = head_;  
}  
}  
}  
template <typename Type, typename Alloc>  
List<Type, Alloc>::List(const List& other)  
: alloc_node_(node_traits::select_on_container_copy_construction(  
other.alloc_node_)) {  
update_head();  
  
BaseNode* now = other.head_;  
Node* other_node;  
Node* node;  
BaseNode* last = head_;  
for (size_t i = 0; i < other.size_; ++i) {  
now = now->next;  
other_node = static_cast<Node*>(now);  
node = node_traits::allocate(alloc_node_, 1);  
try {  
node_traits::construct(alloc_node_, node, other_node->data);  
} catch (...) {  
node_traits::destroy(alloc_node_, node);  
}  
last->next = node;  
node->next = head_;  
++size_;  
  
last = last->next;  
}  
}  
template <typename Type, typename Alloc>  
List<Type, Alloc>& List<Type, Alloc>::operator=(  
const List<Type, Alloc>& other) {  
List copy(other);  
if (!node_traits::propagate_on_container_copy_assignment::value) {  
std::swap(alloc_node_, copy.alloc_node_);  
} else {  
alloc_node_ = other.alloc_node_;  
}  
  
std::swap(copy.size_, size_);  
std::swap(copy.head_, head_);  
  
return *this;  
}  
  
template <typename Type, typename Alloc>  
template <typename... Args>  
void List<Type, Alloc>::insert_after(List<Type, Alloc>::const_iterator iter,  
Args&&... value) {  
Node* node = node_traits::allocate(alloc_node_, 1);  
try {  
node_traits::construct(alloc_node_, node, std::forward<Args>(value)...);  
} catch (...) {  
node_traits::destroy(alloc_node_, node);  
node_traits::deallocate(alloc_node_, node, 1);  
throw;  
}  
BaseNode* base_node = static_cast<BaseNode*>(node);  
BaseNode* position = iter.get_node();  
base_node->next = position->next;  
position->next = base_node;  
  
++size_;  
}  
template <typename Type, typename Alloc>  
void List<Type, Alloc>::erase_after(List<Type, Alloc>::const_iterator iter) {  
BaseNode* position = iter.get_node();  
Node* deleted_object = static_cast<Node*>(position->next);  
position->next = position->next->next;  
deleted_object->next = nullptr;  
  
node_traits::destroy(alloc_node_, deleted_object);  
node_traits::deallocate(alloc_node_, deleted_object, 1);  
  
--size_;  
}  
  
template <typename Type, typename Alloc>  
List<Type, Alloc>::~List() {  
if (head_ == nullptr) {  
return;  
}  
Node* last = static_cast<Node*>(head_->next);  
BaseNode* now = head_;  
for (size_t i = 0; i < size_; ++i) {  
now = last->next;  
node_traits::destroy(alloc_node_, last);  
node_traits::deallocate(alloc_node_, last, 1);  
  
if (i != size_ - 1) {  
last = static_cast<Node*>(now);  
}  
}  
node_traits::deallocate(alloc_node_, reinterpret_cast<Node*>(now), 1);  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::iterator List<Type, Alloc>::begin() {  
return List<Type, Alloc>::iterator(head_->next);  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::iterator List<Type, Alloc>::end() {  
return List<Type, Alloc>::iterator(head_);  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_iterator List<Type, Alloc>::begin() const {  
return List<Type, Alloc>::const_iterator(head_->next);  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_iterator List<Type, Alloc>::end() const {  
return List<Type, Alloc>::const_iterator(head_);  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_iterator List<Type, Alloc>::cbegin() const {  
return List<Type, Alloc>::const_iterator(head_->next);  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_iterator List<Type, Alloc>::cend() const {  
return List<Type, Alloc>::const_iterator(head_);  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::reverse_iterator List<Type, Alloc>::rbegin() {  
return List<Type, Alloc>::reverse_iterator(std::make_reverse_iterator(end()));  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::reverse_iterator List<Type, Alloc>::rend() {  
return List<Type, Alloc>::reverse_iterator(  
std::make_reverse_iterator(begin()));  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_reverse_iterator List<Type, Alloc>::rbegin()  
const {  
return List<Type, Alloc>::const_reverse_iterator(  
std::make_reverse_iterator(end()));  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_reverse_iterator List<Type, Alloc>::rend()  
const {  
return List<Type, Alloc>::const_reverse_iterator(  
std::make_reverse_iterator(begin()));  
}  
  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_reverse_iterator List<Type, Alloc>::crbegin()  
const {  
return List<Type, Alloc>::const_reverse_iterator(  
std::make_reverse_iterator(end()));  
}  
template <typename Type, typename Alloc>  
typename List<Type, Alloc>::const_reverse_iterator List<Type, Alloc>::crend()  
const {  
return List<Type, Alloc>::const_reverse_iterator(  
std::make_reverse_iterator(begin()));  
}  
  
template <typename Type, typename Alloc>  
template <bool IsConst>  
class List<Type, Alloc>::CommonIterator {  
public:  
using difference_type = int64_t;  
using value_type = std::conditional_t<IsConst, const Type, Type>;  
using pointer = value_type*;  
using reference = value_type&;  
reference operator*() const { return static_cast<Node*>(now_)->data; }  
reference get_data() const { return static_cast<Node*>(now_)->data; }  
using iterator_category = std::forward_iterator_tag;  
  
CommonIterator() = default;  
CommonIterator(CommonIterator&& other) : now_(other.now_) {  
other.now_ = nullptr;  
}  
CommonIterator(BaseNode* now) : now_(now) {}  
CommonIterator(const CommonIterator& other) : now_(other.now_) {}  
operator const_iterator() const;  
// operator const_reverse_iterator() const;  
CommonIterator& operator=(const CommonIterator& other) {  
CommonIterator copy = CommonIterator(other.now_);  
std::swap(now_, copy.now_);  
return (*this);  
}  
BaseNode* get_node() const { return now_; }  
  
CommonIterator<IsConst>& operator++();  
List<Type, Alloc>::CommonIterator<IsConst> operator++(int);  
  
bool operator==(  
const List<Type, Alloc>::CommonIterator<IsConst>& other) const;  
  
bool operator!=(  
const List<Type, Alloc>::CommonIterator<IsConst>& other) const;  
~CommonIterator() = default;  
  
private:  
BaseNode* now_;  
};  
// template<typename Type, typename Alloc, bool IsConst>  
// List<Type, Alloc>::CommonIterator<IsConst> &List<Type,  
// Alloc>::CommonIterator<Type, Alloc, IsConst>::operator++() {  
// return <#initializer#>;  
//}  
  
template <typename Type, typename Alloc>  
template <bool IsConst>  
List<Type, Alloc>::CommonIterator<IsConst>::operator const_iterator() const {  
return const_iterator(now_);  
}  
template <typename Type, typename Alloc>  
template <bool IsConst>  
List<Type, Alloc>::template CommonIterator<IsConst>&  
List<Type, Alloc>::CommonIterator<IsConst>::operator++() {  
now_ = now_->next;  
return *this;  
}  
template <typename Type, typename Alloc>  
template <bool IsConst>  
List<Type, Alloc>::template CommonIterator<IsConst>  
List<Type, Alloc>::CommonIterator<IsConst>::operator++(int) {  
CommonIterator<IsConst> copy = (*this);  
++(*this);  
return copy;  
}  
template <typename Type, typename Alloc>  
template <bool IsConst>  
bool List<Type, Alloc>::CommonIterator<IsConst>::operator==(  
const List<Type, Alloc>::CommonIterator<IsConst>& other) const {  
return now_ == other.now_;  
}  
template <typename Type, typename Alloc>  
template <bool IsConst>  
bool List<Type, Alloc>::CommonIterator<IsConst>::operator!=(  
const List<Type, Alloc>::CommonIterator<IsConst>& other) const {  
return !((*this) == other);  
}  
  
template <typename Key,  
typename Value,  
typename Hash = std::hash<Key>,  
typename Equal = std::equal_to<Key>,  
typename Alloc = std::allocator<std::pair<const Key, Value>>>  
class UnorderedMap {  
public:  
using NodeType = std::pair<const Key, Value>;  
struct Node {  
NodeType key_value;  
size_t hash;  
Node() = default;  
Node(const Key& key) : key_value(key, Value()) {  
  
}  
Node(const Key&& key) : key_value(std::move(key), Value()) {}  
Node(Key&& key) : key_value(std::move(key), Value()) {}  
template <typename... Args>  
Node(Args&&... value) : key_value(std::forward<Args>(value)...) {  
  
}  
Node(Node& other) : key_value(other.key_value), hash(other.hash) {}  
Node(Node&& other)  
: key_value(std::move(other.key_value)), hash(other.hash) {  
other.hash = 0;  
}  
// Node(const Key&& key, Value&& value):  
// key_value(std::move(const_cast<Key&&>(key)), std::move(value)) {}  
// Node(Key  
//&&key) : key_value(std::move(key), Value()) {  
// //key = Key();  
// }  
~Node() = default;  
};  
using node_traits =  
typename std::allocator_traits<Alloc>::template rebind_traits<NodeType>;  
using node_alloc =  
typename std::allocator_traits<Alloc>::template rebind_alloc<NodeType>;  
  
[[no_unique_address]] node_alloc alloc_;  
[[no_unique_address]] Equal equal_;  
[[no_unique_address]] Hash hash_;  
  
size_t bucket_count_;  
size_t load_factor_;  
size_t max_load_factor_;  
List<Node, node_alloc> data_;  
  
std::vector<typename List<Node, node_alloc>::iterator> buckets_;  
UnorderedMap()  
: bucket_count_(0),  
max_load_factor_(1),  
buckets_(1, nullptr),  
load_factor_(0),  
data_(alloc_) {}  
UnorderedMap(UnorderedMap&& other)  
: alloc_(  
node_traits::select_on_container_copy_construction(other.alloc_)),  
bucket_count_(other.bucket_count_),  
load_factor_(other.load_factor_),  
max_load_factor_(other.max_load_factor_),  
data_(node_traits::select_on_container_copy_construction(alloc_)),  
buckets_(std::move(other.buckets_)),  
equal_(std::move(other.equal_)),  
hash_(std::move(other.hash_)) {  
other.bucket_count_ = other.load_factor_ = other.max_load_factor_ = 0;  
data_ = (std::move(other.data_));  
// other.data_ = other.buckets_ = alloc_node_ = hash_ = equal_ = nullptr;  
}  
UnorderedMap(const UnorderedMap& other)  
: alloc_(  
node_traits::select_on_container_copy_construction(other.alloc_)),  
bucket_count_(other.bucket_count_),  
load_factor_(other.load_factor_),  
max_load_factor_(other.max_load_factor_),  
data_(node_traits::select_on_container_copy_construction(alloc_)),  
equal_(other.equal_),  
hash_(other.hash_) {  
buckets_.resize(other.buckets_.size());  
auto now_item = data_.end();  
++now_item;  
auto last_item = data_.end();  
int now_bucket = -1;  
while (now_item != data_.end()) {  
if (now_item.get_data().hash % buckets_.size() != now_bucket) {  
now_bucket = now_item.get_data().hash % buckets_.size();  
buckets_[now_bucket] = now_item;  
}  
++now_item;  
++last_item;  
}  
data_ = (other.data_);  
}  
  
template <bool IsConst>  
class CommonIterator;  
  
using iterator = CommonIterator<false>;  
using const_iterator = CommonIterator<true>;  
using reverse_iterator = std::reverse_iterator<iterator>;  
using const_reverse_iterator = std::reverse_iterator<const_iterator>;  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::iterator begin();  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::iterator end();  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator begin() const;  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator end() const;  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator cbegin() const;  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator cend() const;  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::reverse_iterator rbegin();  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::reverse_iterator rend();  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator rbegin()  
const;  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator rend()  
const;  
  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator crbegin()  
const;  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator crend()  
const;  
  
void swap(UnorderedMap& other) {  
std::swap(bucket_count_, other.bucket_count_);  
std::swap(load_factor_, other.load_factor_);  
std::swap(max_load_factor_, other.max_load_factor_);  
std::swap(data_, other.data_);  
std::swap(buckets_, other.buckets_);  
std::swap(alloc_, other.alloc_);  
std::swap(equal_, other.equal_);  
std::swap(hash_, other.hash_);  
}  
UnorderedMap& operator=(UnorderedMap&& other) noexcept {  
UnorderedMap copy = std::move(other);  
swap(copy);  
// copy.bucket_count_ = copy.load_factor_ = copy.max_load_factor_ = 0;  
// copy.data_ = copy.buckets_ = alloc_node_ = hash_ = equal_ = nullptr;  
return *this;  
}  
UnorderedMap& operator=(const UnorderedMap& other) {  
UnorderedMap copy = UnorderedMap(other);  
swap(copy);  
return *this;  
}  
size_t size() { return data_.size(); }  
  
// std::pair<iterator, bool> emplace(std::pair<const Key, Value>&& value) {  
// return emplace(std::);  
// }  
template <typename... Args>  
std::pair<iterator, bool> emplace(Args&&... args) {  
if (load_factor_ >= max_load_factor_) {  
rehash(2 * buckets_.size() + 1);  
}  
  
data_.insert_after(data_.end(), std::forward<Args>(args)...);  
auto new_item = data_.end();  
auto prev_item = new_item;  
++new_item;  
try {  
(*new_item).hash = hash_((*new_item).key_value.first);  
} catch (...) {  
data_.erase_after(data_.end());  
throw;  
}  
auto new_item_hash = (*new_item).hash;  
// std::cout << "hui " << new_item_hash << " " << new_item_hash %  
// buckets_.size() << std::endl;  
if (buckets_[new_item_hash % buckets_.size()] == nullptr) {  
auto next_item = new_item;  
++next_item;  
// std::cout << "huiiiii " << (*new_item).hash << " " << (*new_item).hash  
// % buckets_.size() << std::endl;  
  
buckets_[new_item_hash % buckets_.size()] = data_.end();  
if (((*next_item).hash % buckets_.size()) !=  
new_item_hash % buckets_.size()) {  
buckets_[((*next_item).hash % buckets_.size())] = new_item;  
}  
// std::cout << "huiiiii " << (*new_item).hash << " " << (*new_item).hash  
// % buckets_.size() << std::endl;  
  
update_load_factor();  
return std::make_pair(new_item, true);  
} else {  
auto item_bucket = buckets_[new_item_hash % buckets_.size()];  
++item_bucket;  
if (item_bucket == new_item) {  
++item_bucket;  
}  
while ((item_bucket != data_.end()) &&  
((*item_bucket).hash % buckets_.size() ==  
new_item_hash % buckets_.size())) {  
try {  
if (equal_((*item_bucket).key_value.first,  
(*new_item).key_value.first)) {  
// std::cout << (*item_bucket).key_value.first << " " <<  
// (*new_item).key_value.first << std::endl;  
data_.erase_after(data_.end());  
return {std::move(item_bucket), false};  
}  
} catch (...) {  
throw;  
}  
++item_bucket;  
}  
if (buckets_[new_item_hash % buckets_.size()] != data_.end()) {  
prev_item.get_node()->next = new_item.get_node()->next;  
data_.move_after(buckets_[new_item_hash % buckets_.size()].get_node(),  
new_item.get_node());  
}  
  
auto result = buckets_[new_item_hash % buckets_.size()];  
++result;  
// (*result).hash = new_item_hash;  
update_load_factor();  
return {iterator(std::move(result)), true};  
}  
}  
std::pair<iterator, bool> insert(NodeType&& value) {  
return emplace(std::move(value));  
}  
// std::pair<iterator, bool> insert(const NodeType&& value) {  
// return emplace(std::move(value));  
// }  
template <typename InputIterator>  
void insert(InputIterator&& first, InputIterator&& last) {  
InputIterator now = std::forward<InputIterator>(first);  
while (now != last) {  
// std::cout << (*now).first << std::endl;  
insert((*now));  
++now;  
}  
std::cout << (*now).first << std::endl;  
// insert((*now));  
}  
  
template <typename... Args>  
std::pair<iterator, bool> insert(Args&&... args) {  
return emplace(std::forward<Args>(args)...);  
}  
std::pair<iterator, bool> insert(const NodeType& value) {  
return emplace(value);  
}  
  
void erase(const const_iterator& iter) {  
auto hash = (iter.get_iter_hash()) % buckets_.size();  
std::cout << iter.get_iter_hash() << std::endl;  
auto now_iter = buckets_[hash];  
auto prev_iter = now_iter;  
++now_iter;  
auto next_iter = now_iter;  
++next_iter;  
size_t number_of_items = 0;  
while ((now_iter != data_.end()) &&  
((*now_iter).hash % buckets_.size() == hash)) {  
++number_of_items;  
if (equal_((*now_iter).key_value.first, (*iter).first)) {  
data_.erase_after(prev_iter);  
if ((number_of_items == 1) &&  
((next_iter == data_.end()) || ((*next_iter).hash != hash))) {  
if ((next_iter != data_.end())) {  
buckets_[(*next_iter).hash % buckets_.size()] = prev_iter;  
} else {  
buckets_[hash] = nullptr;  
}  
}  
update_load_factor();  
return;  
}  
++prev_iter;  
++now_iter;  
++next_iter;  
}  
}  
void erase(const iterator& first, const iterator& second) {  
iterator now = iterator(first);  
iterator next = now;  
++next;  
while (now.iter != second.iter) {  
erase(now);  
now = next;  
++next;  
}  
// erase(now);  
}  
  
void rehash(size_t new_size) {  
size_t prev_size = buckets_.size();  
buckets_.resize(new_size);  
  
for (size_t bucket = 0; bucket < prev_size; ++bucket) {  
if (buckets_[bucket] == nullptr) {  
continue;  
}  
auto next = buckets_[bucket];  
auto item = next;  
++next;  
auto after_next = next;  
++after_next;  
// std::cout << (*next).hash << std::endl;  
bool is_something_same = false;  
// std::cout << next.get_node() << " " << std::endl;  
while ((((*next).hash % buckets_.size() == bucket) ||  
((*next).hash % prev_size == bucket)) &&  
(next != data_.end())) {  
if ((*next).hash % buckets_.size() != bucket) {  
item.get_node()->next = after_next.get_node();  
next.get_node()->next = nullptr;  
  
if (buckets_[(*next).hash % buckets_.size()] == nullptr) {  
buckets_[(*next).hash % buckets_.size()] = data_.end();  
data_.move_after(data_.end().get_node(), next.get_node());  
auto last_end_bucket = data_.end();  
++last_end_bucket;  
++last_end_bucket;  
//  
// next.get_node()->next =  
// last_end_bucket.get_node();  
// data_.end().get_node()->next = next.get_node();  
if (last_end_bucket != data_.end()) {  
if (last_end_bucket == after_next) {  
buckets_[(*last_end_bucket).hash % prev_size] = next;  
} else {  
buckets_[(*last_end_bucket).hash % buckets_.size()] = next;  
}  
}  
if (item == data_.end()) {  
item = next;  
}  
} else {  
data_.move_after(  
buckets_[(*next).hash % buckets_.size()].get_node(),  
next.get_node());  
auto new_elem = buckets_[(*next).hash % buckets_.size()];  
++new_elem;  
auto new_prev_elem = new_elem;  
++new_elem;  
if ((*new_elem).hash % buckets_.size() !=  
(*next).hash % buckets_.size()) {  
buckets_[(*new_elem).hash % buckets_.size()] = new_prev_elem;  
}  
}  
  
next = after_next;  
++after_next;  
} else {  
is_something_same = true;  
item = next;  
next = after_next;  
++after_next;  
}  
}  
if (!is_something_same) {  
buckets_[bucket] = nullptr;  
}  
}  
update_load_factor();  
}  
void reserve(size_t new_size) { rehash(new_size); }  
  
iterator find(const Key& key) {  
auto key_hash = hash_(key);  
auto item = buckets_[key_hash % buckets_.size()];  
if (item == nullptr) {  
return end();  
}  
++item;  
while ((typename List<Node, node_alloc>::const_iterator(item) !=  
data_.end()) &&  
((*item).hash % buckets_.size() == key_hash % buckets_.size())) {  
try {  
if (equal_((*item).key_value.first, key)) {  
return iterator(std::move(item));  
}  
} catch (...) {  
throw;  
}  
++item;  
}  
return end();  
}  
const_iterator find(const Key& key) const {  
auto key_hash = hash_(key);  
auto item = buckets_[key_hash % buckets_.size()];  
if (item == nullptr) {  
return end();  
}  
++item;  
while ((typename List<Node, node_alloc>::const_iterator(item) !=  
data_.end()) &&  
((*item).hash % buckets_.size() == key_hash % buckets_.size())) {  
try {  
if (equal_((*item).key_value.first, key)) {  
return const_iterator(std::move(item));  
}  
} catch (...) {  
throw;  
}  
++item;  
}  
return end();  
}  
iterator find(const Key&& key) {  
auto key_hash = hash_(std::move(key));  
auto item = buckets_[key_hash % buckets_.size()];  
if (item == nullptr) {  
return end();  
}  
++item;  
while ((typename List<Node, node_alloc>::const_iterator(item) !=  
(data_.end())) &&  
((*item).hash % buckets_.size() == key_hash % buckets_.size())) {  
try {  
if (equal_((*item).key_value.first, key)) {  
return iterator(std::move(item));  
}  
} catch (...) {  
throw;  
}  
++item;  
}  
return end();  
}  
const_iterator find(const Key&& key) const {  
auto key_hash = hash_(std::move(key));  
auto item = buckets_[key_hash % buckets_.size()];  
if (item == nullptr) {  
return end();  
}  
++item;  
while ((typename List<Node, node_alloc>::const_iterator(item) !=  
(data_.end())) &&  
((*item).hash % buckets_.size() == key_hash % buckets_.size())) {  
try {  
if (equal_((*item).key_value.first, key)) {  
return const_iterator(std::move(item));  
}  
} catch (...) {  
throw;  
}  
++item;  
}  
return end();  
}  
  
Value& at(const Key& key) {  
auto iter = find(key);  
if (iter == end()) {  
throw std::out_of_range("out of range");  
}  
return (*iter).second;  
}  
const Value& at(const Key& key) const {  
auto iter = find(key);  
if (iter == end()) {  
throw std::out_of_range("out of range");  
}  
return (*iter).second;  
}  
Value& at(const Key&& key) {  
auto iter = find(std::move(key));  
if (iter == end()) {  
throw std::out_of_range("out of range");  
}  
return (*iter).second;  
}  
const Value& at(const Key&& key) const {  
auto iter = find(std::move(key));  
if (iter == end()) {  
throw std::out_of_range("out of range");  
}  
return (*iter).second;  
}  
  
Value& operator[](const Key&& key) {  
try {  
return at(std::move(key));  
} catch (...) {  
return (*(emplace(std::move(key)).first)).second;  
}  
}  
Value& operator[](const Key& key) {  
try {  
return at(key);  
} catch (...) {  
return (*(emplace(std::move(key)).first)).second;  
}  
}  
const Value& operator[](const Key&& key) const {  
try {  
return at(key);  
} catch (...) {  
return (*(emplace(std::move(key)).first)).second;  
}  
}  
const Value& operator[](const Key& key) const {  
try {  
return at(key);  
} catch (...) {  
return (*(emplace(std::move(key)).first)).second;  
}  
}  
  
void update_load_factor() { load_factor_ = (data_.size()) / buckets_.size(); }  
};  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
template <bool IsConst>  
class UnorderedMap<Key, Value, Hash, Equal, Alloc>::CommonIterator {  
public:  
using difference_type = int;  
using my_iterator =  
std::conditional_t<IsConst,  
typename List<Node, node_alloc>::const_iterator,  
typename List<Node, node_alloc>::iterator>;  
using value_type = std::conditional_t<IsConst, const NodeType, NodeType>;  
using pointer = value_type*;  
using reference = value_type&;  
using iterator_category = std::forward_iterator_tag;  
operator const_iterator() const { return const_iterator(iter); }  
  
CommonIterator(typename List<Node, node_alloc>::const_iterator&& other)  
: iter(std::move(other)) {}  
CommonIterator(const typename List<Node, node_alloc>::iterator& other)  
: iter(other) {}  
CommonIterator& operator=(const CommonIterator& other) {  
CommonIterator copy = CommonIterator(other);  
std::swap(iter, copy.iter);  
return (*this);  
}  
CommonIterator& operator=(CommonIterator&& other) noexcept {  
CommonIterator copy = std::move(other);  
std::swap(iter, copy.iter);  
return (*this);  
}  
  
CommonIterator& operator++() {  
++iter;  
return (*this);  
}  
CommonIterator operator++(int) {  
CommonIterator copy = CommonIterator(iter++);  
return copy;  
}  
  
CommonIterator(CommonIterator&& other) : iter(std::move(other.iter)) {}  
CommonIterator(const CommonIterator& other) : iter(other.iter) {}  
  
// operator CommonIterator<true>() const {  
// return CommonIterator<true>(iter);  
// }  
bool operator==(const CommonIterator<IsConst>& other) const {  
return iter == other.iter;  
}  
bool operator!=(const CommonIterator<IsConst>& other) const {  
return iter != other.iter;  
}  
size_t get_iter_hash() const { return (*iter).hash; }  
  
reference operator*() const { return iter.get_data().key_value; }  
pointer operator->() const { return &(iter.get_data().key_value); }  
my_iterator iter;  
};  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::begin() {  
return CommonIterator<false>(data_.begin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::end() {  
return CommonIterator<false>(data_.end());  
}  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::begin() const {  
return CommonIterator<true>(data_.begin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::end() const {  
return CommonIterator<true>(data_.end());  
}  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::cbegin() const {  
return UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator(  
data_.cbegin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::cend() const {  
return UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_iterator(  
data_.cend());  
}  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::rbegin() {  
return CommonIterator<false>(data_.rbegin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::rend() {  
return CommonIterator<false>(data_.rend());  
}  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::rbegin() const {  
return CommonIterator<true>(data_.rbegin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::rend() const {  
return CommonIterator<true>(data_.crend());  
}  
  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::crbegin() const {  
return CommonIterator<true>(data_.crbegin());  
}  
template <typename Key,  
typename Value,  
typename Hash,  
typename Equal,  
typename Alloc>  
typename UnorderedMap<Key, Value, Hash, Equal, Alloc>::const_reverse_iterator  
UnorderedMap<Key, Value, Hash, Equal, Alloc>::crend() const {  
return CommonIterator<true>(data_.crend());  
}
```
