# Stoichiometry-Calculator
#Balances chemical equations
#Be sure to view raw text for better formatting

from sympy import Matrix

def main():
    #takes input and puts data into a 2d matrix
    print('Enter an unbalanced chemical equation. (No coefficients.)')
    print('Example input: Fe + H2SO4 = Fe2(SO4)3 + H2')
    equation = input(': ').replace('-->', '=')
    react, prod = equation.replace(' ', '').split('=') #splits the equation into product and reacant sides
    reactants = react.split('+')
    #splits the reactant side into a list of molecules
    products = prod.split('+') 
    #splits product side into a list of molecules
    
    atomList = []
    zeros = [] #populated by zeros to fill in missing values in the final matrix
    matrix = [] 
    i = 0 #the total number of molecules counted so far
    
    for reactant in reactants: #add reacant atoms to matrix as positive integers
        reactant = moleculeSplit(reactant,1) 
        #splits each molecule into a dictionary of atoms
        #key = atomic symbol, value = the quantity of the atom per molecule        
        for atom in reactant:
            if atom in atomList:
                aIndex = atomList.index(atom)
                matrix[aIndex] = populate(matrix[aIndex],i,reactant[atom])
            else:
                atomList.append(atom)
                matrix.append(zeros + [reactant[atom]])
        zeros.append(0)
        i += 1

    for product in products: #add product atoms to matrix as negative integers
        product = moleculeSplit(product,-1)
        for atom in product:
            if atom in atomList:
                aIndex = atomList.index(atom)
                matrix[aIndex] = populate(matrix[aIndex],i,product[atom])
            else:
                atomList.append(atom)
                matrix.append(zeros + [product[atom]])
        zeros.append(0)
        i += 1

    for x in range(len(matrix)):
        while len(matrix[x]) < i: #add zeros to rows to make them even
            matrix[x].append(0)
            
    outPut(nullMath(matrix, i),reactants,products)
    
def moleculeSplit(mole, sign): #splits molecules into atoms
    subScript = ''
    elem = ''
    elements = {}
    mult = 1        #used to multiply numbers in ()
    i = len(mole)
    for i in range(len(mole)-1, -1, -1):
        if mole[i] == ')':
            if subScript == '': subScript = 1
            mult = int(subScript)
            subScript = ''
        elif mole[i] == '(':
            mult = 1
        else:
            char = capLowNum(mole[i]) #returns uppercase, lowercase, or digit
            if char == 'digit':
                subScript = mole[i] + subScript
            else:
                elem = mole[i] + elem
                if char == 'up': #elements begin with capital letters
                    if subScript == '': subScript = 1
                    elements[elem] = int(subScript) * sign * mult
                    subScript = ''
                    elem = ''
    return(elements)
    
def nullMath(matrix, i):
    coEffs = Matrix(matrix).nullspace() #coefficients in decimal form
    notInts = True
    mult = 1
    
    while notInts:
        intCoEffs = []
        for x in range(i):
            if coEffs[0][x] * mult % 1 == 0: #coefficients must be whole numbers
                intCoEffs.append(coEffs[0][x]*mult)
                notInts = False
            else:
                notInts = True
                mult += 1
                break
    return(intCoEffs)

def outPut(coEffs, reactants, products):
    equation = ""
    i = 0
    subscript = str.maketrans("0123456789", "₀₁₂₃₄₅₆₇₈₉")
    for reactant in reactants:
        if coEffs[i] == 1: coEffs[i] = ''
        equation = equation + str(coEffs[i]) + reactant.translate(subscript) + ' + '
        i += 1
    equation = equation[:-2] + " --> "
    for product in products:
        if coEffs[i] == 1: coEffs[i] = ''
        equation = equation + str(coEffs[i]) + product.translate(subscript) + ' + '
        i += 1
    print('')
    print(equation[:-2])
    print('')

def capLowNum(char):
    if ord(char) > 96: return('low')
    if ord(char) < 58: return('digit')
    return('up')

def populate(l, most,new):
    while len(l) < most:
        l.append(0)
    l.append(new)
    return(l)

while True:
    try:
        main()
    except:
        print('')
        print('There appears to be an error with your input. Remember that\
        the first letter of an atomic symbol is always capitalized.\
        Also, check your 0s and Os.')
        print('')
