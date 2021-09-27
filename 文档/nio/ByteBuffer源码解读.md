A container for data of a specific primitive type.
A buffer is a linear, finite sequence of elements of a specific primitive type. Aside from its content, the essential properties of a buffer are its capacity, limit, and position:
A buffer's capacity is the number of elements it contains. The capacity of a buffer is never negative and never changes.
A buffer's limit is the index of the first element that should not be read or written. A buffer's limit is never negative and is never greater than its capacity.
A buffer's position is the index of the next element to be read or written. A buffer's position is never negative and is never greater than its limit.
There is one subclass of this class for each non-boolean primitive type.

