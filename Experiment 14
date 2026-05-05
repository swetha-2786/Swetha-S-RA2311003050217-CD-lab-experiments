class Block:
    def __init__(self, bid, statements=None):
        self.id = bid
        self.stmts = statements or []
        self.successors = []
        self.preds = []

    def __repr__(self):
        return f"B{self.id}"


def link(b1, b2):
    if b2 not in b1.successors:
        b1.successors.append(b2)
    if b1 not in b2.preds:
        b2.preds.append(b1)


def reaching_definitions(blocks):
    all_defs = []
    for b in blocks:
        for i, (dest, _) in enumerate(b.stmts):
            if dest:
                all_defs.append((b.id, i, dest))

    def gen_kill(block):
        gen_b = set()
        kill_b = set()
        defined_in_block = {}
        for i, (dest, _) in enumerate(block.stmts):
            if dest is None:
                continue
            for d in all_defs:
                if d[2] == dest and not (d[0] == block.id and d[1] == i):
                    kill_b.add(d)
            prev = defined_in_block.get(dest)
            if prev is not None:
                gen_b.discard((block.id, prev, dest))
            gen_b.add((block.id, i, dest))
            defined_in_block[dest] = i
        return frozenset(gen_b), frozenset(kill_b)

    GEN = {}
    KILL = {}
    for b in blocks:
        GEN[b.id], KILL[b.id] = gen_kill(b)

    IN = {b.id: set() for b in blocks}
    OUT = {b.id: set(GEN[b.id]) for b in blocks}

    changed = True
    iterations = 0
    while changed:
        changed = False
        iterations += 1
        for b in blocks:
            new_in = set()
            for pred in b.preds:
                new_in |= OUT[pred.id]
            new_out = set(GEN[b.id]) | (new_in - set(KILL[b.id]))
            if new_out != OUT[b.id]:
                changed = True
            IN[b.id] = new_in
            OUT[b.id] = new_out

    return GEN, KILL, IN, OUT, iterations, all_defs


def live_variable_analysis(blocks):
    def use_def(block):
        use_b = set()
        def_b = set()
        for dest, rhs_vars in block.stmts:
            for v in rhs_vars:
                if v not in def_b:
                    use_b.add(v)
            if dest and dest not in use_b:
                def_b.add(dest)
        return frozenset(use_b), frozenset(def_b)

    USE = {}
    DEF = {}
    for b in blocks:
        USE[b.id], DEF[b.id] = use_def(b)

    IN = {b.id: set(USE[b.id]) for b in blocks}
    OUT = {b.id: set() for b in blocks}

    changed = True
    iterations = 0
    while changed:
        changed = False
        iterations += 1
        for b in reversed(blocks):
            new_out = set()
            for succ in b.successors:
                new_out |= IN[succ.id]
            new_in = set(USE[b.id]) | (new_out - set(DEF[b.id]))
            if new_in != IN[b.id] or new_out != OUT[b.id]:
                changed = True
            IN[b.id] = new_in
            OUT[b.id] = new_out

    return USE, DEF, IN, OUT, iterations


def _fmt_defs(defs):
    if not defs:
        return "{}"
    return "{" + ", ".join(f"d{b}{i}({v})" for b, i, v in sorted(defs)) + "}"


def _fmt_vars(vs):
    if not vs:
        return "{}"
    return "{" + ", ".join(sorted(vs)) + "}"


def display_rd(blocks, GEN, KILL, IN, OUT, iters, all_defs):
    print("\n  -- REACHING DEFINITIONS --  (fixed-point after", iters, "iterations)")
    print(f"\n  All definitions: {_fmt_defs(all_defs)}\n")
    print(f"  {'Block':>6}  {'GEN':^30}  {'KILL':^30}  {'IN':^30}  {'OUT':^30}")
    print("  " + "-" * 100)
    for b in blocks:
        print(f"  {'B'+str(b.id):>6}  "
              f"{_fmt_defs(GEN[b.id]):^30}  "
              f"{_fmt_defs(KILL[b.id]):^30}  "
              f"{_fmt_defs(IN[b.id]):^30}  "
              f"{_fmt_defs(OUT[b.id]):^30}")


def display_lv(blocks, USE, DEF, IN, OUT, iters):
    print("\n  -- LIVE VARIABLE ANALYSIS --  (fixed-point after", iters, "iterations)")
    print(f"\n  {'Block':>6}  {'USE':^20}  {'DEF':^20}  {'IN':^20}  {'OUT':^20}")
    print("  " + "-" * 80)
    for b in blocks:
        print(f"  {'B'+str(b.id):>6}  "
              f"{_fmt_vars(USE[b.id]):^20}  "
              f"{_fmt_vars(DEF[b.id]):^20}  "
              f"{_fmt_vars(IN[b.id]):^20}  "
              f"{_fmt_vars(OUT[b.id]):^20}")


def display_cfg(blocks):
    print("\n  -- CONTROL FLOW GRAPH --")
    for b in blocks:
        succs = ", ".join(str(s) for s in b.successors) or "EXIT"
        preds = ", ".join(str(p) for p in b.preds) or "ENTRY"
        print(f"    B{b.id}  preds=[{preds}]  succs=[{succs}]")
        for dest, rhs in b.stmts:
            lhs = dest if dest else "_"
            print(f"         {lhs} = f({', '.join(rhs)})")


def build_example_1():
    B1 = Block(1, [('d', ['a', 'b']), ('e', ['a', 'c'])])
    B2 = Block(2, [('b', ['b', 'd']), ('e', ['a', 'c'])])
    B3 = Block(3, [('d', ['a', 'b'])])
    B4 = Block(4, [(None, ['b', 'e'])])
    link(B1, B2)
    link(B2, B3)
    link(B2, B4)
    link(B3, B2)
    return [B1, B2, B3, B4]


def build_example_2():
    B1 = Block(1, [('x', ['a', 'b']), ('y', ['c'])])
    B2 = Block(2, [('z', ['x', 'y'])])
    B3 = Block(3, [('w', ['z'])])
    link(B1, B2)
    link(B1, B3)
    link(B2, B3)
    return [B1, B2, B3]


def run_analysis(label, blocks):
    print(f"\n{'='*80}")
    print(f"  EXAMPLE: {label}")
    print(f"{'='*80}")
    display_cfg(blocks)
    GEN, KILL, RD_IN, RD_OUT, iters, all_defs = reaching_definitions(blocks)
    display_rd(blocks, GEN, KILL, RD_IN, RD_OUT, iters, all_defs)
    USE, DEF, LV_IN, LV_OUT, iters2 = live_variable_analysis(blocks)
    display_lv(blocks, USE, DEF, LV_IN, LV_OUT, iters2)


if __name__ == "__main__":
    print("=" * 80)
    print("         GLOBAL DATA FLOW ANALYSIS")
    print("         Reaching Definitions + Live Variable Analysis")
    print("=" * 80)

    run_analysis("Dragon Book Classic (loop)", build_example_1())
    run_analysis("Straight-line with branch", build_example_2())

    print("\n  Done.")
