import re

TOKEN_RE = re.compile(
    r'\s*(?:'
    r'(\d+\.\d*|\d*\.\d+|\d+)'
    r'|([A-Za-z_]\w*)'
    r'|([+\-*/^%()])'
    r')'
)


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
    def __init__(self, op, left, right):
        self.op, self.left, self.right = op, left, right


class UnaryOp:
    def __init__(self, op, operand):
        self.op, self.operand = op, operand


class Num:
    def __init__(self, val):
        self.val = val


class Var:
    def __init__(self, name):
        self.name = name


class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0

    def peek(self):
        return self.tokens[self.pos]

    def consume(self, expected=None):
        tok = self.tokens[self.pos]
        if expected and tok[1] != expected:
            raise SyntaxError(f"Expected '{expected}', got '{tok[1]}'")
        self.pos += 1
        return tok

    def parse(self):
        node = self.expr()
        if self.peek()[0] != 'EOF':
            raise SyntaxError(f"Unexpected token: {self.peek()[1]}")
        return node

    def expr(self):
        node = self.term()
        while self.peek()[1] in ('+', '-'):
            op = self.consume()[1]
            node = BinOp(op, node, self.term())
        return node

    def term(self):
        node = self.factor()
        while self.peek()[1] in ('*', '/'):
            op = self.consume()[1]
            node = BinOp(op, node, self.factor())
        return node

    def factor(self):
        tok = self.peek()
        if tok[1] == '(':
            self.consume('(')
            node = self.expr()
            self.consume(')')
            return node
        if tok[1] == '-':
            self.consume('-')
            return UnaryOp('-', self.factor())
        if tok[0] == 'NUM':
            self.consume()
            return Num(tok[1])
        if tok[0] == 'ID':
            self.consume()
            return Var(tok[1])
        raise SyntaxError(f"Unexpected token: {tok[1]}")


def to_postfix(node):
    if isinstance(node, Num):
        return node.val
    if isinstance(node, Var):
        return node.name
    if isinstance(node, UnaryOp):
        return f"{to_postfix(node.operand)} NEG"
    if isinstance(node, BinOp):
        return f"{to_postfix(node.left)} {to_postfix(node.right)} {node.op}"
    raise TypeError(f"Unknown node: {node}")


def to_prefix(node):
    if isinstance(node, Num):
        return node.val
    if isinstance(node, Var):
        return node.name
    if isinstance(node, UnaryOp):
        return f"NEG {to_prefix(node.operand)}"
    if isinstance(node, BinOp):
        return f"{node.op} {to_prefix(node.left)} {to_prefix(node.right)}"
    raise TypeError(f"Unknown node: {node}")


def to_infix(node):
    if isinstance(node, Num):
        return node.val
    if isinstance(node, Var):
        return node.name
    if isinstance(node, UnaryOp):
        return f"(-{to_infix(node.operand)})"
    if isinstance(node, BinOp):
        return f"({to_infix(node.left)} {node.op} {to_infix(node.right)})"
    raise TypeError(f"Unknown node: {node}")


def eval_postfix(postfix_str):
    stack = []
    for tok in postfix_str.split():
        if tok == 'NEG':
            stack.append(-stack.pop())
        elif tok in '+-*/':
            b, a = stack.pop(), stack.pop()
            if tok == '+': stack.append(a + b)
            elif tok == '-': stack.append(a - b)
            elif tok == '*': stack.append(a * b)
            elif tok == '/': stack.append(a / b)
        else:
            stack.append(float(tok))
    return stack[0] if stack else None


def process(expr):
    print(f"\n  Infix    : {expr}")
    try:
        tokens = tokenize(expr)
        tree = Parser(tokens).parse()
        post = to_postfix(tree)
        pre = to_prefix(tree)
        inf = to_infix(tree)
        print(f"  Postfix  : {post}")
        print(f"  Prefix   : {pre}")
        print(f"  Infix(p) : {inf}")
        try:
            val = eval_postfix(post)
            if val is not None:
                print(f"  Value    : {val}")
        except Exception:
            pass
    except SyntaxError as e:
        print(f"  [Error]  : {e}")


if __name__ == "__main__":
    examples = [
        "a + b * c",
        "(a + b) * c",
        "a + b * c - d / e",
        "3 + 5 * 2",
        "(3 + 5) * 2",
        "a * (b + c) * d",
        "-a + b",
        "3 * (4 + 5) - 6 / 2",
    ]

    print("=" * 60)
    print("   INTERMEDIATE CODE GENERATION: Postfix & Prefix")
    print("=" * 60)
    for ex in examples:
        process(ex)

    print("\n" + "=" * 60)
    print("   CUSTOM EXPRESSION")
    print("=" * 60)
    while True:
        try:
            expr = input("\n  Enter infix expression (or 'quit'): ").strip()
            if expr.lower() in ('quit', 'q', 'exit', ''):
                break
            process(expr)
        except KeyboardInterrupt:
            break
    print("\n  Done.")
