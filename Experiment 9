from collections import defaultdict


class Item:
    def __init__(self, lhs, rhs, dot=0):
        self.lhs = lhs
        self.rhs = rhs
        self.dot = dot

    def next_symbol(self):
        if self.dot < len(self.rhs):
            return self.rhs[self.dot]
        return None

    def is_complete(self):
        return self.dot >= len(self.rhs)

    def advance(self):
        return Item(self.lhs, self.rhs, self.dot + 1)

    def __eq__(self, other):
        return (self.lhs == other.lhs and
                self.rhs == other.rhs and
                self.dot == other.dot)

    def __hash__(self):
        return hash((self.lhs, self.rhs, self.dot))

    def __repr__(self):
        rhs = list(self.rhs)
        rhs.insert(self.dot, 'dot')
        return f"[{self.lhs} -> {' '.join(rhs)}]".replace('dot', '\u2022')


class LR0:
    def __init__(self, grammar, start):
        self.grammar = grammar
        self.start = start
        self.aug_start = start + "'"
        self.symbols = set()
        self._collect_symbols()
        self.C = []
        self.goto_table = {}

    def _collect_symbols(self):
        for lhs, prods in self.grammar.items():
            self.symbols.add(lhs)
            for prod in prods:
                for sym in prod:
                    self.symbols.add(sym)

    def closure(self, items):
        closed = set(items)
        worklist = list(items)
        while worklist:
            item = worklist.pop()
            B = item.next_symbol()
            if B and B in self.grammar:
                for prod in self.grammar[B]:
                    new_item = Item(B, tuple(prod), 0)
                    if new_item not in closed:
                        closed.add(new_item)
                        worklist.append(new_item)
        return frozenset(closed)

    def goto(self, items, symbol):
        moved = {item.advance() for item in items
                 if item.next_symbol() == symbol}
        return self.closure(moved)

    def build(self):
        aug_grammar = {self.aug_start: [[self.start]], **self.grammar}
        self.grammar = aug_grammar

        initial = self.closure({Item(self.aug_start, (self.start,), 0)})
        self.C = [initial]
        worklist = [initial]

        while worklist:
            item_set = worklist.pop(0)
            idx = self.C.index(item_set)
            for sym in self.symbols | {self.aug_start}:
                g = self.goto(item_set, sym)
                if not g:
                    continue
                if g not in self.C:
                    self.C.append(g)
                    worklist.append(g)
                self.goto_table[(idx, sym)] = self.C.index(g)

    def display(self):
        print("\n" + "=" * 60)
        print("        CANONICAL COLLECTION OF LR(0) ITEMS")
        print("=" * 60)
        for i, item_set in enumerate(self.C):
            print(f"\n  I{i}:")
            for item in sorted(item_set, key=repr):
                marker = "  *** REDUCE ***" if item.is_complete() else ""
                print(f"    {item}{marker}")

        print("\n" + "=" * 60)
        print("               GOTO TRANSITIONS")
        print("=" * 60)
        print(f"  {'State':>6}  {'Symbol':<15}  {'Next State':>10}")
        print(f"  {'-'*6}  {'-'*15}  {'-'*10}")
        for (state, sym), nxt in sorted(self.goto_table.items()):
            print(f"  {'I'+str(state):>6}  {sym:<15}  {'I'+str(nxt):>10}")


def run_example(name, grammar, start):
    print(f"\n{'#'*60}")
    print(f"  EXAMPLE: {name}")
    print(f"{'#'*60}")
    lr0 = LR0(grammar, start)
    lr0.build()
    lr0.display()


def parse_grammar_input():
    print("\nEnter grammar productions (one per line).")
    print("Format:  A -> B C  |  a b")
    print("Type 'done' when finished.\n")
    grammar = defaultdict(list)
    while True:
        line = input("  Production: ").strip()
        if line.lower() == 'done':
            break
        if '->' not in line:
            print("  [!] Invalid format. Use: A -> B c | d")
            continue
        lhs, rhs_str = line.split('->', 1)
        lhs = lhs.strip()
        for alt in rhs_str.split('|'):
            prod = alt.split()
            grammar[lhs].append(prod)
    start = input("  Start symbol: ").strip()
    return dict(grammar), start


if __name__ == "__main__":
    g1 = {
        'E': [['E', '+', 'T'], ['T']],
        'T': [['T', '*', 'F'], ['F']],
        'F': [['(', 'E', ')'], ['id']]
    }
    run_example("Expression Grammar  E->E+T|T, T->T*F|F, F->(E)|id", g1, 'E')

    g2 = {
        'S': [['C', 'C']],
        'C': [['c', 'C'], ['d']]
    }
    run_example("Simple Grammar  S->CC, C->cC|d", g2, 'S')

    print("\n" + "=" * 60)
    print("         CUSTOM GRAMMAR (LR(0) ITEMS)")
    print("=" * 60)
    try:
        grammar, start = parse_grammar_input()
        if grammar:
            run_example("Custom Grammar", grammar, start)
        else:
            print("  [!] No productions entered.")
    except KeyboardInterrupt:
        print("\n  [!] Cancelled.")
