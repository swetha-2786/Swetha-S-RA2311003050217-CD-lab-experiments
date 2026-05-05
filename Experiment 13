import re


def parse_tac(text):
    instrs = []
    for line in text.strip().splitlines():
        line = line.strip()
        if not line:
            continue
        m = re.match(r'(\w+)\s*=\s*(\w+)\s*([+\-*/])\s*(\w+)', line)
        if m:
            instrs.append(('BIN', m.group(1), m.group(2), m.group(3), m.group(4)))
            continue
        m = re.match(r'(\w+)\s*=\s*-\s*(\w+)', line)
        if m:
            instrs.append(('UNARY', m.group(1), m.group(2)))
            continue
        m = re.match(r'(\w+)\s*=\s*(\w+)$', line)
        if m:
            instrs.append(('COPY', m.group(1), m.group(2)))
            continue
        print(f"  [WARN] Skipping: {line}")
    return instrs


class DAGNode:
    _id_counter = 0

    def __init__(self, op, left=None, right=None, value=None):
        DAGNode._id_counter += 1
        self.id = DAGNode._id_counter
        self.op = op
        self.left = left
        self.right = right
        self.value = value
        self.labels = []

    def __repr__(self):
        if self.op == 'LEAF':
            return f"N{self.id}[{self.value}]"
        if self.right:
            return f"N{self.id}[{self.op}](N{self.left.id}, N{self.right.id})"
        return f"N{self.id}[{self.op}](N{self.left.id})"


class DAGBuilder:
    def __init__(self):
        self.nodes = []
        self.var_map = {}
        self._hash = {}

    def _leaf(self, val):
        if val in self.var_map:
            return self.var_map[val]
        node = DAGNode('LEAF', value=val)
        node.labels.append(val)
        self.nodes.append(node)
        self.var_map[val] = node
        return node

    def _internal(self, op, left, right=None):
        key = (op, left.id, right.id if right else None)
        if key in self._hash:
            return self._hash[key]
        node = DAGNode(op, left, right)
        self.nodes.append(node)
        self._hash[key] = node
        return node

    def _detach(self, res, node):
        if res in self.var_map:
            old = self.var_map[res]
            if res in old.labels:
                old.labels.remove(res)
        if res not in node.labels:
            node.labels.append(res)
        self.var_map[res] = node

    def add(self, instr):
        kind = instr[0]
        if kind == 'COPY':
            _, res, src = instr
            src_node = self.var_map.get(src) or self._leaf(src)
            self._detach(res, src_node)
            return src_node
        if kind == 'BIN':
            _, res, a1, op, a2 = instr
            n1 = self.var_map.get(a1) or self._leaf(a1)
            n2 = self.var_map.get(a2) or self._leaf(a2)
            node = self._internal(op, n1, n2)
            self._detach(res, node)
            return node
        if kind == 'UNARY':
            _, res, a1 = instr
            n1 = self.var_map.get(a1) or self._leaf(a1)
            node = self._internal('UMINUS', n1)
            self._detach(res, node)
            return node
        return None

    def build(self, instructions):
        for instr in instructions:
            self.add(instr)

    def common_subexpressions(self):
        return [n for n in self.nodes if n.op != 'LEAF' and len(n.labels) > 1]

    def optimized_tac(self):
        lines = []
        emitted = set()

        def label_of(node):
            return node.labels[0] if node.labels else f"__t{node.id}"

        def emit(node):
            if node.id in emitted:
                return
            emitted.add(node.id)
            if node.op == 'LEAF':
                return
            if node.left:
                emit(node.left)
            if node.right:
                emit(node.right)
            primary = label_of(node)
            if node.right:
                lines.append(f"  {primary} = {label_of(node.left)} {node.op} {label_of(node.right)}")
            elif node.left:
                lines.append(f"  {primary} = -{label_of(node.left)}")
            for alias in node.labels[1:]:
                lines.append(f"  {alias} = {primary}         # CSE: reuse")

        for node in self.nodes:
            emit(node)
        return lines

    def display(self):
        print("\n  -- DAG NODES --")
        print(f"  {'NodeID':>8}  {'Op':^8}  {'Left':^8}  {'Right':^8}  {'Labels'}")
        print("  " + "-" * 55)
        for n in self.nodes:
            left_s = f"N{n.left.id}" if n.left else "-"
            right_s = f"N{n.right.id}" if n.right else "-"
            labels = ", ".join(n.labels) if n.labels else "-"
            print(f"  {'N'+str(n.id):>8}  {n.op:^8}  {left_s:^8}  {right_s:^8}  {labels}")

        cse = self.common_subexpressions()
        if cse:
            print("\n  -- COMMON SUB-EXPRESSIONS DETECTED --")
            for node in cse:
                print(f"    Node N{node.id} ({node.op}) shared by: {', '.join(node.labels)}")
        else:
            print("\n  -- No common sub-expressions detected --")

        print("\n  -- OPTIMIZED TAC --")
        for line in self.optimized_tac():
            print(line)


def run(label, tac_text):
    print(f"\n{'='*60}")
    print(f"  Example: {label}")
    print(f"{'='*60}")
    print("  Original TAC:")
    for line in tac_text.strip().splitlines():
        print(f"    {line.strip()}")
    instrs = parse_tac(tac_text)
    builder = DAGBuilder()
    builder.build(instrs)
    builder.display()


if __name__ == "__main__":
    ex1 = """
    t1 = a + b
    t2 = a + b
    t3 = t1 * t2
    """

    ex2 = """
    t1 = a + b
    t2 = c * d
    t3 = a + b
    t4 = t1 + t2
    t5 = t3 * t2
    """

    ex3 = """
    t1 = a + b
    t2 = b * c
    t3 = t1 - t2
    """

    print("=" * 60)
    print("      DAG CONSTRUCTION & COMMON SUBEXPRESSION ELIMINATION")
    print("=" * 60)

    run("Classic CSE: a+b computed twice", ex1)
    run("Multiple CSE", ex2)
    run("No CSE", ex3)

    print("\n" + "=" * 60)
    print("  CUSTOM TAC")
    print("=" * 60)
    print("  Enter TAC (e.g. t1 = a + b). Type 'done' when finished.")
    lines = []
    while True:
        try:
            line = input("  > ").strip()
            if line.lower() in ('done', ''):
                break
            lines.append(line)
        except KeyboardInterrupt:
            break
    if lines:
        run("Custom", "\n".join(lines))
    print("\n  Done.")
