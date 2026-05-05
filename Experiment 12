import re


REGISTERS = ['R0', 'R1', 'R2', 'R3']
OP_MAP = {'+': 'ADD', '-': 'SUB', '*': 'MUL', '/': 'DIV'}


def parse_tac(tac_text):
    instructions = []
    for line in tac_text.strip().splitlines():
        line = line.strip()
        if not line:
            continue
        m = re.match(r'(\w+)\s*=\s*(\w+)\s*([+\-*/])\s*(\w+)', line)
        if m:
            instructions.append(('BIN', m.group(1), m.group(2), m.group(3), m.group(4)))
            continue
        m = re.match(r'(\w+)\s*=\s*-\s*(\w+)', line)
        if m:
            instructions.append(('UNARY', m.group(1), m.group(2)))
            continue
        m = re.match(r'(\w+)\s*=\s*(\w+)$', line)
        if m:
            instructions.append(('COPY', m.group(1), m.group(2)))
            continue
        print(f"  [WARN] Unrecognised TAC: {line}")
    return instructions


class CodeGen:
    def __init__(self):
        self.reg_desc = {r: None for r in REGISTERS}
        self.addr_desc = {}
        self.code = []

    def _free_reg(self):
        for r in REGISTERS:
            if self.reg_desc[r] is None:
                return r
        victim_reg = REGISTERS[0]
        victim_var = self.reg_desc[victim_reg]
        if victim_var:
            self.code.append(f"  STORE  {victim_reg}, {victim_var}")
            locs = self.addr_desc.setdefault(victim_var, set())
            locs.add('memory')
        self.reg_desc[victim_reg] = None
        return victim_reg

    def _ensure_in_reg(self, var):
        for r in REGISTERS:
            if self.reg_desc[r] == var:
                return r
        reg = self._free_reg()
        self.code.append(f"  LOAD   {reg}, {var}")
        self.reg_desc[reg] = var
        self.addr_desc.setdefault(var, set()).add(reg)
        return reg

    def _assign_reg(self, var):
        for r in REGISTERS:
            if self.reg_desc[r] == var:
                return r
        reg = self._free_reg()
        old_var = self.reg_desc[reg]
        if old_var:
            locs = self.addr_desc.get(old_var, set())
            locs.discard(reg)
        self.reg_desc[reg] = var
        self.addr_desc.setdefault(var, set()).add(reg)
        return reg

    def gen_bin(self, result, a1, op, a2):
        r1 = self._ensure_in_reg(a1)
        r2 = self._ensure_in_reg(a2)
        res = self._assign_reg(result)
        mnemonic = OP_MAP.get(op, op)
        self.code.append(f"  {mnemonic:<6} {res}, {r1}, {r2}        # {result} = {a1} {op} {a2}")

    def gen_unary(self, result, a1):
        r1 = self._ensure_in_reg(a1)
        res = self._assign_reg(result)
        self.code.append(f"  NEG    {res}, {r1}            # {result} = -{a1}")

    def gen_copy(self, result, a1):
        r1 = self._ensure_in_reg(a1)
        res = self._assign_reg(result)
        self.code.append(f"  MOV    {res}, {r1}            # {result} = {a1}")

    def flush(self):
        self.code.append("")
        self.code.append("  # -- Flush registers --")
        for r in REGISTERS:
            var = self.reg_desc[r]
            if var and 'memory' not in self.addr_desc.get(var, set()):
                self.code.append(f"  STORE  {r}, {var}")

    def generate(self, instructions):
        for instr in instructions:
            kind = instr[0]
            if kind == 'BIN':
                self.gen_bin(instr[1], instr[2], instr[3], instr[4])
            elif kind == 'UNARY':
                self.gen_unary(instr[1], instr[2])
            elif kind == 'COPY':
                self.gen_copy(instr[1], instr[2])
        self.flush()
        return self.code


def run(label, tac_text):
    print(f"\n{'='*60}")
    print(f"  Example: {label}")
    print(f"{'='*60}")
    print("  TAC Input:")
    for line in tac_text.strip().splitlines():
        print(f"    {line.strip()}")
    instructions = parse_tac(tac_text)
    gen = CodeGen()
    code = gen.generate(instructions)
    print("\n  Generated Assembly:")
    for line in code:
        print(line)


if __name__ == "__main__":
    ex1 = """
    t1 = a + b
    t2 = t1 * c
    x  = t2
    """

    ex2 = """
    t1 = b * c
    t2 = a + t1
    t3 = t2 - d
    y  = t3
    """

    ex3 = """
    t1 = a + b
    t2 = c - d
    t3 = t1 * t2
    t4 = e + f
    t5 = t3 / t4
    z  = t5
    """

    run("a + b -> * c -> x", ex1)
    run("a + b*c - d -> y", ex2)
    run("(a+b)*(c-d) / (e+f) -> z", ex3)

    print("\n" + "=" * 60)
    print("  CUSTOM TAC INPUT")
    print("=" * 60)
    print("  Enter TAC lines (e.g.  t1 = a + b). Type 'done' to finish.")
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
