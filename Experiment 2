class State:
    _id = 0
    def __init__(self):
        self.id = State._id
        State._id += 1
        self.transitions = {}
        self.epsilon = []

    def add_transition(self, symbol, state):
        self.transitions.setdefault(symbol, []).append(state)

    def add_epsilon(self, state):
        self.epsilon.append(state)

    @classmethod
    def reset(cls):
        cls._id = 0


class NFA:
    def __init__(self, start, accept):
        self.start = start
        self.accept = accept


def insert_concat(regex):
    result = []
    for i, ch in enumerate(regex):
        result.append(ch)
        if i + 1 < len(regex):
            nxt = regex[i + 1]
            if ch not in ('(', '|') and nxt not in (')', '|', '*', '+', '?'):
                result.append('.')
    return ''.join(result)


def to_postfix(regex):
    prec = {'|': 1, '.': 2, '*': 3, '+': 3, '?': 3}
    out, stk = [], []
    regex = insert_concat(regex)

    for ch in regex:
        if ch == '(':
            stk.append(ch)
        elif ch == ')':
            while stk and stk[-1] != '(':
                out.append(stk.pop())
            if stk:
                stk.pop()
        elif ch in prec:
            while stk and stk[-1] != '(' and stk[-1] in prec and prec[stk[-1]] >= prec[ch]:
                out.append(stk.pop())
            stk.append(ch)
        else:
            out.append(ch)

    while stk:
        out.append(stk.pop())
    return ''.join(out)


def basic_nfa(sym):
    s, a = State(), State()
    s.add_transition(sym, a)
    return NFA(s, a)


def union(n1, n2):
    s, a = State(), State()
    s.add_epsilon(n1.start)
    s.add_epsilon(n2.start)
    n1.accept.add_epsilon(a)
    n2.accept.add_epsilon(a)
    return NFA(s, a)


def concat(n1, n2):
    n1.accept.add_epsilon(n2.start)
    return NFA(n1.start, n2.accept)


def star(n):
    s, a = State(), State()
    s.add_epsilon(n.start)
    s.add_epsilon(a)
    n.accept.add_epsilon(n.start)
    n.accept.add_epsilon(a)
    return NFA(s, a)


def plus_op(n):
    s, a = State(), State()
    s.add_epsilon(n.start)
    n.accept.add_epsilon(n.start)
    n.accept.add_epsilon(a)
    return NFA(s, a)


def optional(n):
    s, a = State(), State()
    s.add_epsilon(n.start)
    s.add_epsilon(a)
    n.accept.add_epsilon(a)
    return NFA(s, a)


def regex_to_nfa(regex):
    State.reset()
    postfix = to_postfix(regex)
    print(f"\nRegex      : {regex}")
    print(f"With concat: {insert_concat(regex)}")
    print(f"Postfix    : {postfix}")

    stk = []
    for ch in postfix:
        if ch == '.':
            b, a = stk.pop(), stk.pop()
            stk.append(concat(a, b))
        elif ch == '|':
            b, a = stk.pop(), stk.pop()
            stk.append(union(a, b))
        elif ch == '*':
            stk.append(star(stk.pop()))
        elif ch == '+':
            stk.append(plus_op(stk.pop()))
        elif ch == '?':
            stk.append(optional(stk.pop()))
        else:
            stk.append(basic_nfa(ch))
    return stk.pop()


def collect_states(nfa):
    visited, queue, states = set(), [nfa.start], []
    while queue:
        s = queue.pop(0)
        if s.id in visited:
            continue
        visited.add(s.id)
        states.append(s)
        for sl in s.transitions.values():
            queue.extend(x for x in sl if x.id not in visited)
        queue.extend(x for x in s.epsilon if x.id not in visited)
    return sorted(states, key=lambda x: x.id)


def display_nfa(nfa):
    states = collect_states(nfa)
    symbols = sorted({sym for s in states for sym in s.transitions})

    print(f"\nStart: q{nfa.start.id}   Accept: q{nfa.accept.id}")
    header = f"{'State':<10}"
    for sym in symbols:
        header += f"  {sym:<12}"
    header += f"  {'epsilon':<15}"
    print(header)
    print("-" * len(header))

    for s in states:
        mark = ""
        if s == nfa.start: mark += "->"
        if s == nfa.accept: mark += "*"
        row = f"{mark}q{s.id:<8}"
        for sym in symbols:
            targets = s.transitions.get(sym, [])
            row += f"  {','.join(f'q{t.id}' for t in targets) if targets else '-':<12}"
        eps = s.epsilon
        row += f"  {','.join(f'q{t.id}' for t in eps) if eps else '-':<15}"
        print(row)
    print(f"\nTotal states: {len(states)}, Symbols: {{{', '.join(symbols)}}}")


def main():
    print("RE to NFA Converter (Thompson's Construction)")
    print("-" * 45)

    examples = ["ab", "a|b", "a*", "(a|b)*", "ab|cd", "(a|b)*abb", "a(b|c)*d"]
    for r in examples:
        nfa = regex_to_nfa(r)
        display_nfa(nfa)
        print()

    print("\nEnter a regex ('exit' to quit):")
    while True:
        try:
            r = input(">>> ").strip()
        except EOFError:
            break
        if r.lower() == 'exit':
            break
        if r:
            try:
                display_nfa(regex_to_nfa(r))
            except Exception as e:
                print(f"Error: {e}")


if __name__ == "__main__":
    main()
