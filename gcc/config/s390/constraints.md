;; Constraints definitions belonging to the gcc backend for IBM S/390.
;; Copyright (C) 2006-2022 Free Software Foundation, Inc.
;; Written by Wolfgang Gellerich, using code and information found in
;; files s390.md, s390.h, and s390.c.
;;
;; This file is part of GCC.
;;
;; GCC is free software; you can redistribute it and/or modify it under
;; the terms of the GNU General Public License as published by the Free
;; Software Foundation; either version 3, or (at your option) any later
;; version.
;;
;; GCC is distributed in the hope that it will be useful, but WITHOUT ANY
;; WARRANTY; without even the implied warranty of MERCHANTABILITY or
;; FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
;; for more details.
;;
;; You should have received a copy of the GNU General Public License
;; along with GCC; see the file COPYING3.  If not see
;; <http://www.gnu.org/licenses/>.


;;
;; Special constraints for s/390 machine description:
;;
;;    a -- Any address register from 1 to 15.
;;    b -- Memory operand whose address is a symbol reference or a symbol
;;         reference + constant which can be proven to be naturally aligned.
;;    c -- Condition code register 33.
;;    d -- Any register from 0 to 15.
;;    f -- Floating point registers.
;;    j -- Multiple letter constraint for constant scalar and vector values
;;         j00: constant zero scalar or vector
;;         jm1: constant scalar or vector with all bits set
;;         jxx: contiguous bitmask of 0 or 1 in all vector elements
;;         jyy: constant consisting of byte chunks being either 0 or 0xff
;;         jKK: constant vector with all elements having the same value and
;;              matching K constraint
;;         jm6: An integer operand with the lowest order 6 bits all ones.
;;         jdd: A constant operand that fits into the data section.
;;         j>f: An integer operand whose lower 32 bits are greater than or equal to 15
;;         jb4: An unsigned constant 4 bit operand.
;;    t -- Access registers 36 and 37.
;;    v -- Vector registers v0-v31.
;;    C -- A signed 8-bit constant (-128..127)
;;    D -- An unsigned 16-bit constant (0..65535)
;;    G -- Const double zero operand
;;    I -- An 8-bit constant (0..255).
;;    J -- A 12-bit constant (0..4095).
;;    K -- A 16-bit constant (-32768..32767).
;;    L -- Value appropriate as displacement.
;;         (0..4095) for short displacement
;;         (-524288..524287) for long displacement
;;    M -- Constant integer with a value of 0x7fffffff.
;;    N -- Multiple letter constraint followed by 4 parameter letters.
;;         0..9,x:  number of the part counting from most to least significant
;;         S,H,Q:   mode of the part
;;         D,S,H:   mode of the containing operand
;;         0,F:     value of the other parts (F - all bits set)
;;         --
;;         xxDq     satisfies s390_contiguous_bitmask_p for DImode
;;                  (with possible wraparound of the one-bit range)
;;         xxSw     satisfies s390_contiguous_bitmask_p for SImode
;;                  (with possible wraparound of the one-bit range)
;;         xxSq     satisfies s390_contiguous_bitmask_nowrap_p for SImode
;;                  (without wraparound of the one-bit range)
;;
;;         The constraint matches if the specified part of a constant
;;         has a value different from its other parts.  If the letter x
;;         is specified instead of a part number, the constraint matches
;;         if there is any single part with non-default value.
;;    O -- Multiple letter constraint followed by 1 parameter.
;;         s:  Signed extended immediate value (-2G .. 2G-1).
;;         p:  Positive extended immediate value (0 .. 4G-1).
;;         n:  Negative extended immediate value (-4G+1 .. -1).
;;         These constraints do not accept any operand if the machine does
;;         not provide the extended-immediate facility.
;;    P -- Any integer constant that can be loaded without literal pool.
;;    Q -- Memory reference without index register and with short displacement.
;;    R -- Memory reference with index register and short displacement.
;;    S -- Memory reference without index register but with long displacement.
;;    T -- Memory reference with index register and long displacement.
;;    A -- Multiple letter constraint followed by Q, R, S, or T:
;;         Offsettable memory reference of type specified by second letter.
;;    B -- Multiple letter constraint followed by Q, R, S, or T:
;;         Memory reference of the type specified by second letter that
;;         does *not* refer to a literal pool entry.
;;    U -- Pointer with short displacement. (deprecated - use ZR)
;;    W -- Pointer with long displacement. (deprecated - use ZT)
;;    Y -- Address style operand without index.
;;    ZQ -- Pointer without index register and with short displacement.
;;    ZR -- Pointer with index register and short displacement.
;;    ZS -- Pointer without index register but with long displacement.
;;    ZT -- Pointer with index register and long displacement.
;;    ZL -- LARL operand when in 64-bit mode, otherwise nothing.
;;
;;


;;
;;  Register constraints.
;;

(define_register_constraint "a"
  "ADDR_REGS"
  "Any address register from 1 to 15.")


(define_register_constraint "c"
  "CC_REGS"
  "Condition code register 33")


(define_register_constraint "d"
  "GENERAL_REGS"
  "Any register from 0 to 15")


(define_register_constraint "f"
  "FP_REGS"
  "Floating point registers")


(define_register_constraint "t"
  "ACCESS_REGS"
  "@internal
   Access registers 36 and 37")


(define_register_constraint "v"
  "VEC_REGS"
  "Vector registers v0-v31")


;;
;;  General constraints for constants.
;;

(define_constraint "C"
  "@internal
   An 8-bit signed immediate constant (-128..127)"
  (and (match_code "const_int")
       (match_test "ival >= -128 && ival <= 127")))


(define_constraint "D"
  "An unsigned 16-bit constant (0..65535)"
  (and (match_code "const_int")
       (match_test "ival >= 0 && ival <= 65535")))


(define_constraint "G"
  "@internal
   Const double zero operand"
   (and (match_code "const_double")
        (match_test "s390_float_const_zero_p (op)")))


(define_constraint "I"
  "An 8-bit constant (0..255)"
  (and (match_code "const_int")
       (match_test "(unsigned HOST_WIDE_INT) ival <= 255")))


(define_constraint "J"
  "A 12-bit constant (0..4095)"
  (and (match_code "const_int")
       (match_test "(unsigned HOST_WIDE_INT) ival <= 4095")))


(define_constraint "K"
  "A 16-bit constant (-32768..32767)"
  (and (match_code "const_int")
       (match_test "ival >= -32768 && ival <= 32767")))


(define_constraint "L"
  "Value appropriate as displacement.
      (0..4095) for short displacement
      (-524288..524287) for long displacement"
  (and (match_code "const_int")
       (match_test "TARGET_LONG_DISPLACEMENT ?
              (ival >= -524288 && ival <= 524287)
            : (ival >= 0 && ival <= 4095)")))


(define_constraint "M"
  "Constant integer with a value of 0x7fffffff"
  (and (match_code "const_int")
       (match_test "ival == 2147483647")))


(define_constraint "P"
  "@internal
   Any integer constant that can be loaded without literal pool"
   (and (match_code "const_int")
        (match_test "legitimate_reload_constant_p (GEN_INT (ival))")))


(define_address_constraint "Y"
  "Address style operand without index register"

;; Simply check for base + offset style operands.  Reload will take
;; care of making sure we have a proper base register.

  (match_test "s390_decompose_addrstyle_without_index (op, NULL, NULL)"  ))


;; Shift count operands are not necessarily legitimate addresses
;; but the predicate shift_count_operand will only allow
;; proper operands.  If reload/lra need to change e.g. a spilled register
;; they can still do so via the special handling of address constraints.
;; To avoid further reloading (caused by a non-matching constraint) we
;; always return true here as the predicate's checks are already sufficient.

(define_address_constraint "jsc"
  "Address style operand used as shift count."
  (match_test "true" ))


;;    N -- Multiple letter constraint followed by 4 parameter letters.
;;         0..9,x:  number of the part counting from most to least significant
;;         S,H,Q:   mode of the part
;;         D,S,H:   mode of the containing operand
;;         0,F:     value of the other parts (F = all bits set)
;;
;;         The constraint matches if the specified part of a constant
;;         has a value different from its other parts.  If the letter x
;;         is specified instead of a part number, the constraint matches
;;         if there is any single part with non-default value.
;;
;; The following patterns define only those constraints that are actually
;; used in s390.md.  If you need an additional one, simply add it in the
;; obvious way.  Function s390_N_constraint_str is ready to handle all
;; combinations.
;;


(define_constraint "NxQS0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"xQS0\", ival)")))


(define_constraint "NxHD0"
  "@internal"
   (and (match_code "const_int")
        (match_test "s390_N_constraint_str (\"xHD0\", ival)")))


(define_constraint "NxSD0"
  "@internal"
   (and (match_code "const_int")
        (match_test "s390_N_constraint_str (\"xSD0\", ival)")))


(define_constraint "NxQD0"
  "@internal"
   (and (match_code "const_int")
        (match_test "s390_N_constraint_str (\"xQD0\", ival)")))


(define_constraint "N3HD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"3HD0\", ival)")))


(define_constraint "N2HD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"2HD0\", ival)")))


(define_constraint "N1SD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1SD0\", ival)")))


(define_constraint "N1HS0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1HS0\", ival)")))


(define_constraint "N1HD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1HD0\", ival)")))


(define_constraint "N0SD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0SD0\", ival)")))


(define_constraint "N0HS0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0HS0\", ival)")))


(define_constraint "N0HD0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0HD0\", ival)")))


(define_constraint "NxQDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"xQDF\", ival)")))


(define_constraint "N1SDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1SDF\", ival)")))


(define_constraint "N0SDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0SDF\", ival)")))


(define_constraint "N3HDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"3HDF\", ival)")))


(define_constraint "N2HDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"2HDF\", ival)")))


(define_constraint "N1HDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1HDF\", ival)")))


(define_constraint "N0HDF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0HDF\", ival)")))


(define_constraint "N0HSF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"0HSF\", ival)")))


(define_constraint "N1HSF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"1HSF\", ival)")))


(define_constraint "NxQSF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"xQSF\", ival)")))


(define_constraint "NxQHF"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"xQHF\", ival)")))


(define_constraint "NxQH0"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_N_constraint_str (\"xQH0\", ival)")))

(define_constraint "NxxDw"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_contiguous_bitmask_p (ival, true, 64, NULL, NULL)")))

(define_constraint "NxxSq"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_contiguous_bitmask_p (ival, false, 32, NULL, NULL)")))

(define_constraint "NxxSw"
  "@internal"
  (and (match_code "const_int")
       (match_test "s390_contiguous_bitmask_p (ival, true, 32, NULL, NULL)")))

;;
;; Double-letter constraints starting with O follow.
;;


(define_constraint "Os"
  "@internal
   Signed extended immediate value (-2G .. 2G-1).
   This constraint will only match if the machine provides
   the extended-immediate facility."
  (and (match_code "const_int")
       (match_test "s390_O_constraint_str ('s', ival)")))


(define_constraint "Op"
  "@internal
   Positive extended immediate value (0 .. 4G-1).
   This constraint will only match if the machine provides
   the extended-immediate facility."
  (and (match_code "const_int")
       (match_test "s390_O_constraint_str ('p', ival)")))


(define_constraint "On"
  "@internal
   Negative extended immediate value (-4G+1 .. -1).
   This constraint will only match if the machine provides
   the extended-immediate facility."
  (and (match_code "const_int")
       (match_test "s390_O_constraint_str ('n', ival)")))


;;
;; Vector and scalar constraints for constant values follow.
;;

(define_constraint "j00"
  "Zero scalar or vector constant"
  (match_test "op == CONST0_RTX (GET_MODE (op))"))

(define_constraint "jm1"
  "All one bit scalar or vector constant"
  (match_test "op == CONSTM1_RTX (GET_MODE (op))"))

; vector generate mask operand - support for up to 64 bit elements
(define_constraint "jxx"
  "@internal"
  (and (match_code "const_vector")
       (match_test "s390_contiguous_bitmask_vector_p (op, NULL, NULL)")))

; vector generate byte mask operand - this is only supposed to deal
; with real vectors 128 bit values of being either 0 or -1 are handled
; with j00 and jm1
(define_constraint "jyy"
  "@internal"
  (and (match_code "const_vector")
       (match_test "s390_bytemask_vector_p (op, NULL)")))

; vector replicate immediate operand - support for up to 64 bit elements
(define_constraint "jKK"
  "@internal"
  (and (and (and (match_code "const_vector")
		 (match_test "const_vec_duplicate_p (op)"))
	    (match_test "GET_MODE_UNIT_SIZE (GET_MODE (op)) <= 8"))
       (match_test "satisfies_constraint_K (XVECEXP (op, 0, 0))")))

(define_constraint "jm6"
  "@internal An integer operand with the lowest order 6 bits all ones."
  (match_operand 0 "const_int_6bitset_operand"))

(define_constraint "j>f"
  "@internal An integer operand whose lower 32 bits are greater than or equal to 15."
  (and (match_code "const_int")
       (match_test "(unsigned int)(ival & 0xffffffff) >= 15")))

(define_constraint "jb4"
  "@internal Constant unsigned integer 4 bit value"
  (and (match_code "const_int")
       (match_test "ival >= 0 && ival <= 15")))

;;
;; Memory constraints follow.
;;

(define_memory_constraint "Q"
  "Memory reference without index register and with short displacement"
  (match_test "s390_mem_constraint (\"Q\", op)"))


(define_memory_constraint "R"
  "Memory reference with index register and short displacement"
  (match_test "s390_mem_constraint (\"R\", op)"))


(define_memory_constraint "S"
  "Memory reference without index register but with long displacement"
  (match_test "s390_mem_constraint (\"S\", op)"))


(define_memory_constraint "T"
  "Memory reference with index register and long displacement"
  (match_test "s390_mem_constraint (\"T\", op)"))


(define_memory_constraint "b"
  "Memory reference whose address is a naturally aligned symbol reference."
  (match_test "MEM_P (op)
               && s390_check_symref_alignment (XEXP (op, 0),
                                               GET_MODE_SIZE (GET_MODE (op)))"))

; This defines 'm' as normal memory constraint.  This is only possible
; since the standard memory constraint is re-defined in s390.h using
; the TARGET_MEM_CONSTRAINT macro.
(define_memory_constraint "m"
  "Matches the most general memory address for pre-z10 machines."
  (match_test "s390_mem_constraint (\"T\", op)"))

(define_memory_constraint "AQ"
  "@internal
   Offsettable memory reference without index register and with short displacement"
  (match_test "s390_mem_constraint (\"AQ\", op)"))


(define_memory_constraint "AR"
  "@internal
   Offsettable memory reference with index register and short displacement"
  (match_test "s390_mem_constraint (\"AR\", op)"))


(define_memory_constraint "AS"
  "@internal
   Offsettable memory reference without index register but with long displacement"
  (match_test "s390_mem_constraint (\"AS\", op)"))


(define_memory_constraint "AT"
  "@internal
   Offsettable memory reference with index register and long displacement"
  (match_test "s390_mem_constraint (\"AT\", op)"))



(define_constraint "BQ"
  "@internal
   Memory reference without index register and with short
   displacement that does *not* refer to a literal pool entry."
  (match_test "s390_mem_constraint (\"BQ\", op)"))


(define_constraint "BR"
  "@internal
   Memory reference with index register and short displacement that
   does *not* refer to a literal pool entry. "
  (match_test "s390_mem_constraint (\"BR\", op)"))


(define_constraint "BS"
  "@internal
   Memory reference without index register but with long displacement
   that does *not* refer to a literal pool entry. "
  (match_test "s390_mem_constraint (\"BS\", op)"))


(define_constraint "BT"
  "@internal
   Memory reference with index register and long displacement that
   does *not* refer to a literal pool entry. "
  (match_test "s390_mem_constraint (\"BT\", op)"))


(define_address_constraint "U"
  "Pointer with short displacement. (deprecated - use ZR)"
  (match_test "s390_mem_constraint (\"ZR\", op)"))

(define_address_constraint "W"
  "Pointer with long displacement. (deprecated - use ZT)"
  (match_test "s390_mem_constraint (\"ZT\", op)"))


(define_address_constraint "ZQ"
  "Pointer without index register and with short displacement."
  (match_test "s390_mem_constraint (\"ZQ\", op)"))

(define_address_constraint "ZR"
  "Pointer with index register and short displacement."
  (match_test "s390_mem_constraint (\"ZR\", op)"))

(define_address_constraint "ZS"
  "Pointer without index register but with long displacement."
  (match_test "s390_mem_constraint (\"ZS\", op)"))

(define_address_constraint "ZT"
  "Pointer with index register and long displacement."
  (match_test "s390_mem_constraint (\"ZT\", op)"))

(define_constraint "ZL"
  "LARL operand when in 64-bit mode, otherwise nothing."
  (match_test "TARGET_64BIT && larl_operand (op, VOIDmode)"))

;; This constraint must behave like "i", in particular, the matching values
;; must never be placed into registers or memory by
;; cfgexpand.c:expand_asm_stmt.  It could be straightforward to start its name
;; with a letter from genpreds.c:const_int_constraints, however it would
;; require using (match_code "const_int"), which is infeasible.  To achieve the
;; same effect, that is, setting maybe_allows_reg and maybe_allows_mem to false
;; in genpreds.c:add_constraint, we explicitly exclude reg, subreg and mem
;; codes.
(define_constraint "jdd"
  "A constant operand that fits into the data section.
   Usage of this constraint might produce a relocation."
  (and (not (match_code "reg"))
       (not (match_code "subreg"))
       (not (match_code "mem"))
       (match_test "CONSTANT_P (op)")))
