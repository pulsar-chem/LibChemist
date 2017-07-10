foreach(name TestAtom TestAtomicInfo TestBasisSet TestBasisShell
             TestBasisSetParser TestSetOfAtoms TestSetOfAtomsParser)
    NEW_TEST(${name} UnitTests)
endforeach()
