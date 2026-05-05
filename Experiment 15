class StaticAllocator:
    def __init__(self, base=0):
        self.base = base
        self.current = base
        self.table = {}

    def allocate(self, name, size, var_type="int"):
        if name in self.table:
            raise ValueError(f"Static variable '{name}' already allocated.")
        addr = self.current
        self.table[name] = (addr, size, var_type)
        self.current += size
        return addr

    def address_of(self, name):
        if name not in self.table:
            raise KeyError(f"'{name}' not in static area.")
        return self.table[name][0]

    def display(self):
        print("\n  -- STATIC ALLOCATION TABLE --")
        print(f"  {'Variable':>12}  {'Type':>8}  {'Address':>10}  {'Size':>6}")
        print("  " + "-" * 45)
        for name, (addr, size, typ) in sorted(self.table.items(), key=lambda x: x[1][0]):
            print(f"  {name:>12}  {typ:>8}  {addr:>10}  {size:>6}")
        print(f"\n  Total static segment: {self.current - self.base} bytes  "
              f"({self.base}..{self.current - 1})")


class ActivationRecord:
    def __init__(self, func_name, return_addr, params):
        self.func_name = func_name
        self.return_addr = return_addr
        self.params = dict(params)
        self.locals = {}

    def __repr__(self):
        return f"AR[{self.func_name}]  params={self.params}, locals={self.locals}"


class StackAllocator:
    def __init__(self):
        self.stack = []
        self.sp = 0

    def call(self, func_name, params, return_addr="RET"):
        ar = ActivationRecord(func_name, return_addr, params)
        self.stack.append(ar)
        self.sp += 1
        print(f"  [CALL ]  {func_name}({params})  depth={self.sp}")
        return ar

    def alloc_local(self, name, value=None):
        if not self.stack:
            raise RuntimeError("Stack is empty.")
        ar = self.stack[-1]
        ar.locals[name] = value
        print(f"  [LOCAL]  {ar.func_name}: {name} = {value}")

    def set_local(self, name, value):
        self.stack[-1].locals[name] = value

    def get_local(self, name):
        ar = self.stack[-1]
        if name in ar.locals:
            return ar.locals[name]
        if name in ar.params:
            return ar.params[name]
        raise KeyError(f"'{name}' not found in current AR.")

    def ret(self):
        if not self.stack:
            raise RuntimeError("Stack underflow!")
        ar = self.stack.pop()
        self.sp -= 1
        print(f"  [RETRN]  {ar.func_name}  depth={self.sp}")
        return ar

    def display(self):
        print("\n  -- CALL STACK (top = most recent) --")
        if not self.stack:
            print("  (empty)")
            return
        for i, ar in reversed(list(enumerate(self.stack))):
            print(f"  Frame {i}: {ar}")


class HeapBlock:
    def __init__(self, start, size, free=True, tag=None):
        self.start = start
        self.size = size
        self.free = free
        self.tag = tag

    def __repr__(self):
        status = "FREE" if self.free else f"USED({self.tag})"
        return f"[{self.start}..{self.start+self.size-1}] {status}"


class HeapAllocator:
    def __init__(self, capacity=256):
        self.capacity = capacity
        self.blocks = [HeapBlock(0, capacity, free=True)]
        self.alloc_map = {}

    def malloc(self, size, tag=None):
        best = None
        for blk in self.blocks:
            if blk.free and blk.size >= size:
                if best is None or blk.size < best.size:
                    best = blk
        if best is None:
            raise MemoryError(f"Heap: cannot allocate {size} bytes.")
        idx = self.blocks.index(best)
        leftover = best.size - size
        best.size = size
        best.free = False
        best.tag = tag
        if leftover > 0:
            self.blocks.insert(idx + 1, HeapBlock(best.start + size, leftover, free=True))
        self.alloc_map[tag] = best
        print(f"  [MALLOC] tag={tag} size={size} -> addr={best.start}")
        return best.start

    def free(self, tag):
        if tag not in self.alloc_map:
            raise KeyError(f"Heap: unknown tag '{tag}'.")
        blk = self.alloc_map.pop(tag)
        blk.free = True
        blk.tag = None
        print(f"  [FREE  ] tag={tag} addr={blk.start} size={blk.size}")
        self._coalesce()

    def _coalesce(self):
        merged = True
        while merged:
            merged = False
            for i in range(len(self.blocks) - 1):
                a, b = self.blocks[i], self.blocks[i + 1]
                if a.free and b.free:
                    a.size += b.size
                    del self.blocks[i + 1]
                    merged = True
                    break

    def display(self):
        print("\n  -- HEAP MEMORY MAP --")
        print(f"  Total capacity: {self.capacity} bytes")
        print(f"  {'Block':>6}  {'Start':>6}  {'End':>6}  {'Size':>6}  {'Status'}")
        print("  " + "-" * 45)
        for i, blk in enumerate(self.blocks):
            end = blk.start + blk.size - 1
            status = "FREE" if blk.free else f"ALLOC (tag={blk.tag})"
            print(f"  {i:>6}  {blk.start:>6}  {end:>6}  {blk.size:>6}  {status}")
        free_bytes = sum(b.size for b in self.blocks if b.free)
        used_bytes = self.capacity - free_bytes
        print(f"\n  Used: {used_bytes} bytes   Free: {free_bytes} bytes")


def demo_static():
    print("\n" + "=" * 60)
    print("  1. STATIC ALLOCATION")
    print("=" * 60)
    sa = StaticAllocator(base=1000)
    sa.allocate("counter", 4, "int")
    sa.allocate("pi", 8, "double")
    sa.allocate("flag", 1, "bool")
    sa.allocate("buffer", 64, "char[]")
    sa.display()
    print(f"\n  Address of 'pi'     : {sa.address_of('pi')}")
    print(f"  Address of 'buffer' : {sa.address_of('buffer')}")


def demo_stack():
    print("\n" + "=" * 60)
    print("  2. STACK ALLOCATION  (Factorial example)")
    print("=" * 60)
    stk = StackAllocator()

    stk.call("main", {})
    stk.alloc_local("n", 3)

    stk.call("factorial", [("n", 3)], return_addr="main+12")
    stk.alloc_local("result", None)
    stk.display()

    stk.call("factorial", [("n", 2)], return_addr="factorial+20")
    stk.alloc_local("result", None)

    stk.call("factorial", [("n", 1)], return_addr="factorial+20")
    stk.alloc_local("result", 1)
    stk.display()

    print("\n  -- Unwinding stack --")
    stk.ret()
    stk.ret()
    stk.display()
    stk.ret()
    stk.ret()
    stk.display()


def demo_heap():
    print("\n" + "=" * 60)
    print("  3. HEAP ALLOCATION  (Best-Fit Free List)")
    print("=" * 60)
    heap = HeapAllocator(capacity=128)

    heap.malloc(20, tag="A")
    heap.malloc(15, tag="B")
    heap.malloc(30, tag="C")
    heap.display()

    print()
    heap.free("B")
    heap.malloc(10, tag="D")
    heap.display()

    print()
    heap.free("A")
    heap.free("D")
    heap.display()


if __name__ == "__main__":
    print("=" * 60)
    print("     STORAGE ALLOCATION STRATEGIES")
    print("     Static  |  Stack  |  Heap")
    print("=" * 60)

    demo_static()
    demo_stack()
    demo_heap()

    print("\n" + "=" * 60)
    print("  INTERACTIVE HEAP DEMO")
    print("=" * 60)
    heap = HeapAllocator(128)
    print("  Commands:  alloc <tag> <size>  |  free <tag>  |  show  |  quit")
    while True:
        try:
            cmd = input("\n  heap> ").strip().split()
            if not cmd:
                continue
            if cmd[0] == 'quit':
                break
            elif cmd[0] == 'alloc' and len(cmd) == 3:
                heap.malloc(int(cmd[2]), tag=cmd[1])
            elif cmd[0] == 'free' and len(cmd) == 2:
                heap.free(cmd[1])
            elif cmd[0] == 'show':
                heap.display()
            else:
                print("  Unknown command.")
        except (KeyboardInterrupt, EOFError):
            break
        except Exception as e:
            print(f"  Error: {e}")
    print("\n  Done.")
