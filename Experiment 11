import re
from itertools import count


TOKEN_RE = re.compile(
    r'\s*(?:(\d+\.\d*|\d*\.\d+|\d+)|([A-Za-z_]\w*)|([+\-*/^%%()]))')


def tokenize(expr):
    tokens = []
    for m in TOKEN_RE.finditer(expr):
        if m.group(1):
            tokens.append(('NUM', m.group(1)))
        elif m.group(2):
            tokens.append(('ID', m.group(2)))
        elif m.group(3):
            tokens.append(('OP', m.group(3)))
    tokens.append(('EOF', ''))
    return tokens


class BinOp:
    def __init__(self, op, l, r):
        self.op, self.l, self.r = op, l, r


class Unary:
    def __init__(self, op, v):
        self.op, self.v = op, v


class Leaf:
    def __init__(self, v):
        self.v = v


class Parser:
    def __init__(self, toks):
        self.t, self.p = toks, 0

    def peek(self):
        return self.t[self.p]

    def eat(self):
        tok = self.t[self.p]
        self.p += 1
        return tok

    def parse(self):
        return self.expr()

    def expr(self):
        n = self.term()
        while self.peek()[1] in ('+', '-'):
            op = self.eat()[1]
            n = BinOp(op, n, self.term())
        return n

    def term(self):
        n = self.factor()
        while self.peek()[1] in ('*', '/'):
            op = self.eat()[1]
            n = BinOp(op, n, self.factor())
        return n

    def factor(self):
        tok = self.peek()
        if tok[1] == '(':
            self.eat()
            n = self.expr()
            self.eat()
            return n
        if tok[1] == '-':
            self.eat()
            return Unary('-', self.factor())
        self.eat()
        return Leaf(tok[1])


class TACGen:
    def __init__(self):
        self._ctr = count(1)
        self.instructions = []

    def new_temp(self):
        return f"t{next(self._ctr)}"

    def emit(self, op, arg1='', arg2='', result=''):
        self.instructions.append((op, arg1, arg2, result))
        return result

    def gen(self, node):
        if isinstance(node, Leaf):
            return node.v
        if isinstance(node, Unary):
            v = self.gen(node.v)
            t = self.new_temp()
            self.emit('UMINUS', v, '', t)
            return t
        if isinstance(node, BinOp):
            l = self.gen(node.l)
            r = self.gen(node.r)
            t = self.new_temp()
            self.emit(node.op, l, r, t)
            return t
        return ''


def print_quadruples(instructions):
    print("\n  -- QUADRUPLES  (op, arg1, arg2, result) --")
    header = f"  {'#':>4}  {'op':^8}  {'arg1':^8}  {'arg2':^8}  {'result':^8}"
    print(header)
    print("  " + "-" * (len(header) - 2))
    for i, (op, a1, a2, res) in enumerate(instructions):
        print(f"  {i:>4}  {op:^8}  {a1:^8}  {a2:^8}  {res:^8}")


def print_triples(instructions):
    print("\n  -- TRIPLES  (index, op, arg1, arg2) --")
    header = f"  {'#':>4}  {'op':^8}  {'arg1':^10}  {'arg2':^10}"
    print(header)
    print("  " + "-" * (len(header) - 2))

    def fmt(val):
        for j, (_, _, _, res) in enumerate(instructions):
            if res == val:
                return f"({j})"
        return val

    for i, (op, a1, a2, _) in enumerate(instructions):
        print(f"  {i:>4}  {op:^8}  {fmt(a1):^10}  {fmt(a2):^10}")


def print_indirect_triples(instructions):
    print("\n  -- INDIRECT TRIPLES  (pointer array + triple table) --")

    def fmt(val):
        for j, (_, _, _, res) in enumerate(instructions):
            if res == val:
                return f"({j})"
        return val

    triple_table = {}
    for i, (op, a1, a2, _) in enumerate(instructions):
        triple_table[i] = (op, fmt(a1), fmt(a2))

    pointer_array = list(range(len(instructions)))

    print("\n  Pointer Array:")
    print(f"  {'Ptr#':>6}  {'-> Triple#':>10}")
    print("  " + "-" * 18)
    for pi, ti in enumerate(pointer_array):
        print(f"  {pi:>6}  {ti:>10}")

    print("\n  Triple Table:")
    print(f"  {'Triple#':>8}  {'op':^8}  {'arg1':^10}  {'arg2':^10}")
    print("  " + "-" * 40)
    for ti, (op, a1, a2) in triple_table.items():
        print(f"  {ti:>8}  {op:^8}  {a1:^10}  {a2:^10}")


def process(expr):
    print(f"\n{'='*60}")
    print(f"  Expression: {expr}")
    print(f"{'='*60}")
    try:
        tokens = tokenize(expr)
        tree = Parser(tokens).parse()
        gen = TACGen()
        gen.gen(tree)
        instrs = gen.instructions
        print_quadruples(instrs)
        print_triples(instrs)
        print_indirect_triples(instrs)
    except Exception as e:
        print(f"  [Error]: {e}")


if __name__ == "__main__":
    examples = [
        "a + b * c",
        "(a + b) * (c - d)",
        "a * b + c * d - e",
        "a + b * c - d / e + f",
    ]

    print("=" * 60)
    print("  INTERMEDIATE CODE: Quadruple / Triple / Indirect Triple")
    print("=" * 60)

    for ex in examples:
        process(ex)

    print("\n" + "=" * 60)
    print("  CUSTOM EXPRESSION")
    print("=" * 60)
    while True:
        try:
            expr = input("\n  Enter expression (or 'quit'): ").strip()
            if expr.lower() in ('quit', 'q', 'exit', ''):
                break
            process(expr)
        except (KeyboardInterrupt, EOFError):
            break
    print("\n  Done.")
