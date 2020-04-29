# DecimalFormat导致线程阻塞的问题

 - [应用场景](#应用场景)
 
 - [线上出现问题](#线上出现问题)
 
 - [分析DecimalFormat源码](#分析DecimalFormat源码)
 
 - [解决方法](#解决方法)
 
 ### 应用场景
     DecimalFormat是取小数精度的，线上实际应用场景是模型计算的时候保留小数位数的时候用到。
 
 ### 线上出现问题
     因为我们对没有个项目都有服务监控，通过CAT项目飘红定位具体项目,通过后台项目监控查看到线程阻塞,并且持续一段时间一直没有下降
     并且是间歇性的。于是就通过运维同时的协助通过快照记录下来当时出现问题的情况。然后对出现的问题进行分析。可以查看到当时就是由
     于DecimalFormat导致线程阻塞的。
  
 ### 分析DecimalFormat源码
         /**
          * Formats a double to produce a string.
          * @param number    The double to format
          * @param result    where the text is to be appended
          * @param delegate notified of locations of sub fields
          * @exception       ArithmeticException if rounding is needed with rounding
          *                  mode being set to RoundingMode.UNNECESSARY
          * @return The formatted number string
          */
         private StringBuffer format(double number, StringBuffer result,
                                     FieldDelegate delegate) {
             if (Double.isNaN(number) ||
                (Double.isInfinite(number) && multiplier == 0)) {
                 int iFieldStart = result.length();
                 result.append(symbols.getNaN());
                 delegate.formatted(INTEGER_FIELD, Field.INTEGER, Field.INTEGER,
                                    iFieldStart, result.length(), result);
                 return result;
             }
     
             /* Detecting whether a double is negative is easy with the exception of
              * the value -0.0.  This is a double which has a zero mantissa (and
              * exponent), but a negative sign bit.  It is semantically distinct from
              * a zero with a positive sign bit, and this distinction is important
              * to certain kinds of computations.  However, it's a little tricky to
              * detect, since (-0.0 == 0.0) and !(-0.0 < 0.0).  How then, you may
              * ask, does it behave distinctly from +0.0?  Well, 1/(-0.0) ==
              * -Infinity.  Proper detection of -0.0 is needed to deal with the
              * issues raised by bugs 4106658, 4106667, and 4147706.  Liu 7/6/98.
              */
             boolean isNegative = ((number < 0.0) || (number == 0.0 && 1/number < 0.0)) ^ (multiplier < 0);
     
             if (multiplier != 1) {
                 number *= multiplier;
             }
     
             if (Double.isInfinite(number)) {
                 if (isNegative) {
                     append(result, negativePrefix, delegate,
                            getNegativePrefixFieldPositions(), Field.SIGN);
                 } else {
                     append(result, positivePrefix, delegate,
                            getPositivePrefixFieldPositions(), Field.SIGN);
                 }
     
                 int iFieldStart = result.length();
                 result.append(symbols.getInfinity());
                 delegate.formatted(INTEGER_FIELD, Field.INTEGER, Field.INTEGER,
                                    iFieldStart, result.length(), result);
     
                 if (isNegative) {
                     append(result, negativeSuffix, delegate,
                            getNegativeSuffixFieldPositions(), Field.SIGN);
                 } else {
                     append(result, positiveSuffix, delegate,
                            getPositiveSuffixFieldPositions(), Field.SIGN);
                 }
     
                 return result;
             }
     
             if (isNegative) {
                 number = -number;
             }
     
             // at this point we are guaranteed a nonnegative finite number.
             assert(number >= 0 && !Double.isInfinite(number));
     
             synchronized(digitList) {
                 int maxIntDigits = super.getMaximumIntegerDigits();
                 int minIntDigits = super.getMinimumIntegerDigits();
                 int maxFraDigits = super.getMaximumFractionDigits();
                 int minFraDigits = super.getMinimumFractionDigits();
     
                 digitList.set(isNegative, number, useExponentialNotation ?
                               maxIntDigits + maxFraDigits : maxFraDigits,
                               !useExponentialNotation);
                 return subformat(result, delegate, isNegative, false,
                            maxIntDigits, minIntDigits, maxFraDigits, minFraDigits);
             }
         }
         
         分析源码可以看出方法内部用synchronized修饰，能保证线程的安全性，但是是阻塞的,在多线程中会导致线程阻塞。
  
 ### 解决方法
     自己封装一下，保证每个线程都会有一个DecimalFormat对象的clone,在多线程中不会出现阻塞的情况。
        
        public class ThreadLocalDecimalFormat extends ThreadLocal<DecimalFormat> {
            DecimalFormat proto;
        
            public ThreadLocalDecimalFormat(String pattern) {
                DecimalFormat tmp = new DecimalFormat(pattern);
                this.proto = tmp;
            }
        
            protected DecimalFormat initialValue() {
                return (DecimalFormat)this.proto.clone();
            }
        }
       
> reubenwang@foxmail.com
> 没事别找我，找我也不在！--我很忙🦆