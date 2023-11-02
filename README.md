//@version=5
indicator('SEL ARMI', scale=scale.right, overlay=true)
//****************************** INPUTS *****************************
angle       = input.float(defval=0,minval=-1,maxval=1, step=0.05,title="Angle")
armi_len    = input.float(defval=9.6, minval=0.6,step=0.2, title='RMA Lenght')
armi_mom    = input.float(defval=1.2, minval=0.6, step=0.2, title='RMI Lookback')
highval     = input.float(defval=0.6, minval=0,maxval=1,step=0.05, title='High Trigger')
lowval      = input.float(defval=-0.6, minval=-1,maxval=0, step=0.05, title='Low Trigger')

TRENDING(y, a) =>
    float x = na
    b = a == 0 ? 1 : a / math.abs(a)
    if b == 1
        x := (2 * math.pow((y + 1) / 2, -b * math.pow(a - b, -b)) - 1 + -(2 * math.pow((-y + 1) / 2, b * (b - a)) - 1)) / 2
    if b == -1
        x := (2 * math.pow((y + 1) / 2, -b * (a - b)) - 1 + -(2 * math.pow((-y + 1) / 2, b * math.pow(b - a, b)) - 1)) / 2
    x
RMA(SERIES, LENGHT) =>
    ALPHA = 1 / LENGHT
    float SUM = na
    SUM := ALPHA * SERIES + (1.0 - ALPHA) * nz(SUM[1])
    SUM
ARMI(SOURCE, LENGHT, MOMENTUM) =>
    INC = RMA(math.max(SOURCE - RMA(SOURCE, MOMENTUM * 2), 0), LENGHT)
    DEC = RMA(math.max(RMA(SOURCE, MOMENTUM * 2) - SOURCE, 0), LENGHT)
    RMI = DEC == 0.0 ? 0.0 : (INC - DEC) / (INC + DEC)
    RMI
COLOR(SOURCE) =>
	RED = 255 * ((1+SOURCE)/2)
	GRE = 255 * ((1-math.abs(SOURCE))*0.5)
	BLU = 255 * ((math.abs(SOURCE-1)/2))
	TRA = 0//255 * ((1-math.abs(SOURCE))*0.5)
	OUT = color.rgb(RED,GRE,BLU,TRA)


plot(TRENDING(ARMI(close, armi_len, armi_mom),angle),color=COLOR(TRENDING(ARMI(close, armi_len, armi_mom),angle)))
plot(0)
plot(highval,color=color.red)
plot(lowval,color=color.blue)

alertcondition(TRENDING(ARMI(close, armi_len, armi_mom),angle) > highval,"High A.RMI", "H_A.RMI")
alertcondition(TRENDING(ARMI(close, armi_len, armi_mom),angle) < lowval,"Low A.RMI", "L_A.RMI")
