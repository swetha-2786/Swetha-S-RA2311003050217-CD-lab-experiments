def epsilon_closure(states, transitions):
    closure = set(states)
    stack = list(states)
    while stack:
        s = stack.pop()
        if s in transitions and 'e' in transitions[s]:
            for ns in transitions[s]['e']:
                if ns not in closure:
                    closure.add(ns)
                    stack.append(ns)
    return frozenset(closure)


def move(states, symbol, transitions):
    result = set()
    for s in states:
        if s in transitions and symbol in transitions[s]:
            result.update(transitions[s][symbol])
    return result


def nfa_to_dfa(states, alphabet, transitions, start, accept):
    dfa_start = epsilon_closure({start}, transitions)
    dfa_states = [dfa_start]
    unmarked = [dfa_start]
    dfa_trans = {}
    names = {dfa_start: 'A'}
    name_id = ord('B')

    print("\nSubset Construction Steps:")
    print(f"  e-closure({{{start}}}) = {set(dfa_start)} => {names[dfa_start]}")

    while unmarked:
        cur = unmarked.pop(0)
        dfa_trans[cur] = {}
        for sym in sorted(alphabet):
            mv = move(cur, sym, transitions)
            if not mv:
                dfa_trans[cur][sym] = frozenset()
                continue
            ns = epsilon_closure(mv, transitions)
            print(f"  d({names[cur]}, {sym}) = e-closure({set(mv)}) = {set(ns)}", end="")
            if ns not in dfa_states:
                names[ns] = chr(name_id)
                name_id += 1
                dfa_states.append(ns)
                unmarked.append(ns)
                print(f" => NEW {names[ns]}")
            else:
                print(f" => {names[ns]}")
            dfa_trans[cur][sym] = ns

    dfa_accept = [ds for ds in dfa_states if ds & set(accept)]
    return dfa_states, alphabet, dfa_trans, dfa_start, dfa_accept, names


def show_nfa(states, alphabet, transitions, start, accept):
    print(f"\nNFA Transition Table:")
    syms = sorted(alphabet) + ['e']
    header = f"{'State':<10}" + "".join(f"{s:<15}" for s in syms)
    print(header)
    print("-" * len(header))
    for s in sorted(states):
        mark = ("-> " if s == start else "") + ("* " if s in accept else "")
        row = f"{mark}{s:<8}"
        for sym in syms:
            t = transitions.get(s, {}).get(sym, set())
            row += f"{'{' + ','.join(sorted(t)) + '}' if t else '-':<15}"
        print(row)


def show_dfa(dfa_states, alphabet, dfa_trans, dfa_start, dfa_accept, names):
    print(f"\nDFA Transition Table:")
    header = f"{'DFA':<10}{'NFA States':<25}" + "".join(f"{s:<12}" for s in sorted(alphabet))
    print(header)
    print("-" * len(header))
    for ds in dfa_states:
        mark = ("-> " if ds == dfa_start else "") + ("* " if ds in dfa_accept else "")
        nfa_str = "{" + ",".join(sorted(ds)) + "}"
        row = f"{mark}{names[ds]:<8}{nfa_str:<25}"
        for sym in sorted(alphabet):
            t = dfa_trans[ds].get(sym, frozenset())
            row += f"{names.get(t, '-'):<12}"
        print(row)
    print(f"\nStart: {names[dfa_start]}")
    print(f"Accept: {{{', '.join(names[s] for s in dfa_accept)}}}")
    print(f"Total DFA states: {len(dfa_states)}")


def main():
    print("NFA to DFA Converter (Subset Construction)")
    print("-" * 45)

    print("\n--- Example 1: (a|b)*abb ---")
    t1 = {'q0': {'a': {'q0','q1'}, 'b': {'q0'}}, 'q1': {'b': {'q2'}}, 'q2': {'b': {'q3'}}}
    show_nfa({'q0','q1','q2','q3'}, {'a','b'}, t1, 'q0', {'q3'})
    show_dfa(*nfa_to_dfa({'q0','q1','q2','q3'}, {'a','b'}, t1, 'q0', {'q3'}))

    print("\n--- Example 2: a*b (with epsilon transitions) ---")
    t2 = {'q0': {'a': {'q0'}, 'e': {'q1'}}, 'q1': {'b': {'q2'}}}
    show_nfa({'q0','q1','q2'}, {'a','b'}, t2, 'q0', {'q2'})
    show_dfa(*nfa_to_dfa({'q0','q1','q2'}, {'a','b'}, t2, 'q0', {'q2'}))

    print("\nEnter your own NFA? (y/n): ", end="")
    try:
        if input().strip().lower() == 'y':
            st = [s.strip() for s in input("States (comma sep): ").split(',')]
            al = [s.strip() for s in input("Alphabet (comma sep): ").split(',')]
            start = input("Start state: ").strip()
            acc = [s.strip() for s in input("Accept states (comma sep): ").split(',')]
            tr = {}
            print("Transitions (from,symbol,to) - type 'done' to finish:")
            print("  Use 'e' for epsilon")
            while True:
                t = input("  > ").strip()
                if t.lower() == 'done':
                    break
                parts = [p.strip() for p in t.split(',')]
                if len(parts) != 3:
                    print("  Bad format")
                    continue
                f, sym, to = parts
                tr.setdefault(f, {}).setdefault(sym, set()).add(to)
            show_nfa(set(st), set(al), tr, start, set(acc))
            show_dfa(*nfa_to_dfa(set(st), set(al), tr, start, set(acc)))
    except EOFError:
        pass


if __name__ == "__main__":
    main()
