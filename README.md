# VSD — Week 1: Day 3 — Introduction to Optimization

## Combinational Optimization

1. **Constant propagation**
2. **Boolean simplification** using K‑maps or Quine–McCluskey

---

## Sequential Logic Optimization

**Basic:**

* Sequential constant propagation

**Advanced:**

* State optimization
* Retiming

**Example:** If D=0 in a DFF, then Q will never be 1. The 0 propagates down the circuit so the DFF can be completely removed, reducing gates and flip‑flops. If `set` (async) is connected, optimization is not possible because Q is sync and set is async. Optimization happens only when output is constant.

**Advanced:**

* State optimization: remove unused states.
* Cloning: reduce delay by duplicating registers closer to loads.
* Retiming: push delay from one circuit to another to reduce t_max and increase clock speed.

---

## LAB

### Combinational

1. **opt_check.v** simplified to `y = ab`.

   * `opt_clean -purge` is used for optimizations.
   * ![img1](https://github.com/user-attachments/assets/176b1a70-c7b2-4d6f-be47-ae456d24cee7)

2. **opt_check2.v** — we get AND–OR gate.

   * ![img2](https://github.com/user-attachments/assets/53a81129-1ee2-4f7b-8c92-b6a5979022c3)

3. **opt_check3.v** — expected: `y = abc`

   * ![img3](https://github.com/user-attachments/assets/d744e502-e8cb-4836-bf2b-a98b101c650e)

4. Example 4

   * ![img4](https://github.com/user-attachments/assets/12263af4-bac6-4ead-b0b8-28c077a7a190)

5. **multiple_module_opt.v**

   * ![img5](https://github.com/user-attachments/assets/dcd680a4-810f-4d9a-91cb-c2cd575577d7)

6. Example 6

   * ![img6](https://github.com/user-attachments/assets/3ad075a5-f107-4cbc-a6c5-1f902b94b88c)

---

### Sequential

1. **Dff-const1** — `d=1`, `rst=1` → Q stays 0; after `rst=0` Q becomes 1 → no optimization.

   * ![img7](https://github.com/user-attachments/assets/64f65a86-eaa1-46ec-a74e-333cc1a69de1)
   * ![img8](https://github.com/user-attachments/assets/206a9eb8-2000-4b6b-ab75-f24e243b9647)

2. **Dff-const2** — `d=1`, `rst(set)=1` → Q=1; after `rst(set)=0` Q stays 1 → optimized.

   * ![img9](https://github.com/user-attachments/assets/9b7fafd0-6b59-436d-a344-171c8025f99e)
   * ![img10](https://github.com/user-attachments/assets/4d4b8334-09a2-4e5b-9fbb-8dd6a85fd8f2)

3. **Dff-const3** — neither ff1 nor ff2 can be optimized.

   * ![img11](https://github.com/user-attachments/assets/e032df7b-63a2-430f-9cb3-b74b7a17de66)

4. **Dff-const4** - Optimization Ocourrs
  * ![img12](https://github.com/user-attachments/assets/0bb5b8d9-c867-4d68-9781-4d4de9894f75)
  * ![img13](https://github.com/user-attachments/assets/da627113-e455-4d37-82a1-b8ce01a3e262)

5. **Dff-const5** - q change so no optimization
  * ![img14](https://github.com/user-attachments/assets/7a0c916d-5b84-46e4-baf0-1b645086cb9f)
  * ![img15](https://github.com/user-attachments/assets/8988f37d-add7-4a8a-8171-7aa3d0dff0f6)

## Unused Output Optimization 

1. **UpCounter (3bit)**
   we are only usinf the q[0] the other outputs are unused thus they neednot be present in the design.q=count[0] -> depend on msb only.
   In q=count[2:0]==3'b100 depend on all the bits.
   In case 1 the bit is toggled in all cycle.-> one flop is enough which we see in the synthesis report. the dff output is take and fed back into the d which toggles it evervy cycle.
   Any LOGIC that doesnt used all the outputs is OPTIMIZED.
   In case 2 three flop is needed which we see in the synthesis report. So the Output is not Optimized. 
