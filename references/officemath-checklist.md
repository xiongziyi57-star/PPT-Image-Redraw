# OfficeMath Checklist

Use this reference when a PPT image-redraw task includes mathematical symbols or equations.

## Formula Candidates

Treat these as PowerPoint equations, not normal text:

- Functions: `f(x)`, `s(y_t,k)`, `q_b(k)`, `π_t(k)`
- Subscripts: `μ_c`, `c_t`, `y_t`, `q_b`, `π_t`
- Greek letters: `μ`, `π`, `λ`, `α`, `β`
- Probabilities and distributions
- Piecewise definitions and cases
- Softmax formulas

If a symbol is part of a Chinese sentence, split the sentence and insert the symbol as a small equation object at the correct coordinate.

## Required XML Pattern

PowerPoint may not render raw `m:oMathPara` inside an `a:p`. Wrap math in `a14:m`.

```xml
<a:p>
  <a:pPr algn="ctr">
    <a:defRPr sz="1050">
      <a:latin typeface="Cambria Math"/>
      <a:ea typeface="Cambria Math"/>
      <a:cs typeface="Cambria Math"/>
    </a:defRPr>
  </a:pPr>
  <a14:m>
    <m:oMathPara>
      <m:oMath>...</m:oMath>
    </m:oMathPara>
  </a14:m>
</a:p>
```

The slide root must include:

```xml
xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math"
xmlns:a14="http://schemas.microsoft.com/office/drawing/2010/main"
```

## Common OOXML Nodes

- Plain math text: `<m:r><m:t>f(x)</m:t></m:r>`
- Subscript: `<m:sSub><m:e>base</m:e><m:sub>sub</m:sub></m:sSub>`
- Piecewise brace: `<m:d><m:dPr><m:begChr m:val="{"/><m:endChr m:val=""/></m:dPr>...</m:d>`
- Multi-line cases: `<m:eqArr><m:e>case 1</m:e><m:e>case 2</m:e></m:eqArr>`

## Regression Checks

Read `ppt/slides/slide1.xml` from the generated PPTX and assert:

- `officeDocument/2006/math` exists
- `schemas.microsoft.com/office/drawing/2010/main` exists
- `slide.count("<a14:m>")` equals the expected formula object count
- `slide.count("<m:oMathPara>")` equals the expected formula object count
- `slide.count("<m:sSub>")` is nonzero when subscripts are present
- `slide.count("<m:eqArr>")` is nonzero for piecewise formulas
- Plain text leftovers such as `<a:t>f(x)</a:t>`, `<a:t>qb(k)</a:t>`, or `<a:t>πt(k)</a:t>` are absent
