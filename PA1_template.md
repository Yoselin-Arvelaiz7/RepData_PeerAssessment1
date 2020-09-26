Read data
=========

Vamos a leer los datos usando la libreria `readr` y usaremos la función
`read_csv()`

    library(readr)
    activity<-read_csv('activity.csv',col_types=cols(steps = col_double(),
      date = col_date(format = ""),
      interval = col_double()
    ))
    head(activity)

    # A tibble: 6 x 3
      steps date       interval
      <dbl> <date>        <dbl>
    1    NA 2012-10-01        0
    2    NA 2012-10-01        5
    3    NA 2012-10-01       10
    4    NA 2012-10-01       15
    5    NA 2012-10-01       20
    6    NA 2012-10-01       25

First Question: What is mean total number of steps taken per day?
=================================================================

1.  Calculate the total number of steps taken per day.

Vamos a calcular el número total de pasos por dia. Usaremos la función
`sum()`.

    total_steps=sum(activity$steps,na.rm = TRUE)
    total_steps

    [1] 570608

La suma total de números de pasos por día es 5.7060810^{5}

1.  Make a histogram of the total number of steps taken each day

Usaremos la libreria `ggplot2` para crear el histograma del total de
pasos por dia. La cantidad de intervalos es `bins=60` y el tamaño es por
dia, es decir, `binwidth=1`.

    library(ggplot2)
    ggplot(data=activity,aes(activity$date))+
      geom_histogram(aes(weight=activity$steps),bins=60,binwidth = 1)+ylab('Steps')+xlab('Day')

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

1.  Calculate and report the mean and median of the total number of
    steps taken per day

Vamos a calcular la media y la mediana del numero de pasos totales por
dia. Para eso, usaremos las funciones `mean()` y `median()` e ignoremos
los valores faltantes (missing values) usando el parámetro `na.rm=TRUE`

    media<-mean(activity$steps,na.rm = TRUE)
    mediana<-median(activity$steps,na.rm = TRUE)
    media

    [1] 37.3826

    mediana

    [1] 0

Media y la mediana respectiva son 37.38 y 0

Second Question: What is the average daily activity pattern?
============================================================

1.  Make a time series plot (type = “l”) of the 5-minute interval
    (x-axis) and the average number of steps taken, averaged across all
    days (y-axis)

En el siguiente código vamos a calcular el promedio de los pasos por dia
tomados de acuerdo al intervalo.

    filas<-dim(activity)[1]
    promedio<-c()
    for (i in 1:288){
      z<-c()
      for (j in seq(i,filas,by=288)){
        z<-c(z,activity$steps[j])
      }
      promedio<-c(promedio,mean(z,na.rm = TRUE))
    }
    head(promedio)

    [1] 1.7169811 0.3396226 0.1320755 0.1509434 0.0754717 2.0943396

Para graficar la serie de tiempo, vamos a crear un dataframe con los
intervalos y el vector promedio. Usaremos la función `cbind()` para unir
los vectores y la función `data.frame()` para convertir los datos en
dataframe.

    datos<-data.frame(cbind(activity$interval[1:288],promedio))
    colnames(datos)<-c('interval','promedio')
    head(datos)

      interval  promedio
    1        0 1.7169811
    2        5 0.3396226
    3       10 0.1320755
    4       15 0.1509434
    5       20 0.0754717
    6       25 2.0943396

Usaremos `geom_line()` para graficar la serie de tiempo.

    ggplot(data=datos,aes(datos$interval,datos$promedio))+geom_line()+ylab('Average steps')+ xlab('Intervals 5 min')

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

1.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

Queremos saber cuál es el intervalo que posee mayor promedio de pasos
por dia. Para eso, usaremos la función `which.max()`

    max_inter<-datos$interval[which.max(datos$promedio)]
    max_inter

    [1] 835

El intervalo con mayor pasos en promedio por dia es el intervalo 835

Third Question: Imputing missing values
=======================================

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

Veamos cuantos valores faltantes hay en los datos activity. Usaremos las
funciones `which()` y `is.na()`.

    valores_faltantes<-length(which(is.na(activity)))
    valores_faltantes

    [1] 2304

Hay 2304 valores faltantes (missing values).

1.  Devise a strategy for filling in all of the missing values in the
    dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

La estrategia que voy a utilzar para rellenar los valores faltantes es
el rellenado con la mediana que es 0.

1.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

Vamos ahora a rellanar los valores faltantes con la mediana.

    filas<-dim(activity)[1]
    for (i in 1:filas){
      if (is.na(activity$steps[i])){
        activity$steps[i]<-mediana
      }
    }

    head(activity)

    # A tibble: 6 x 3
      steps date       interval
      <dbl> <date>        <dbl>
    1     0 2012-10-01        0
    2     0 2012-10-01        5
    3     0 2012-10-01       10
    4     0 2012-10-01       15
    5     0 2012-10-01       20
    6     0 2012-10-01       25

1.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day. Do these values differ from the estimates from the first
    part of the assignment? What is the impact of imputing missing data
    on the estimates of the total daily number of steps?

Vamos a graficar el histograma ahora con los datos sin valores
faltantes.

    library(ggplot2)
    ggplot(data=activity,aes(activity$date))+
      geom_histogram(aes(weight=activity$steps),bins=60,binwidth = 1)+ylab('Steps')+xlab('Day')

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-11-1.png)

Ahora calculemos la media y la mediana.

    media<-mean(activity$steps)
    mediana<-median(activity$steps)
    media

    [1] 32.47996

    mediana

    [1] 0

La media y la mediana son respectivamente 32.48 y 0. Podemos observar
que la mediana no influyó ya que los valores que faltaban fueron
rellenados por la mediana, pero la media bajo un poco de 37 a 32. Los
valores faltantes siempre influyen en los datos y cuando se rellenan se
tiene que conocer muy bien los datos para armar una buena estrategia que
sume importante información.

Last Question: Are there differences in activity patterns between weekdays and weekends?
========================================================================================

1.  Create a new factor variable in the dataset with two levels –
    “weekday” and “weekend” indicating whether a given date is a weekday
    or weekend day.

Para ver si hay una diferencia de patron en los dias laborables o fin de
semanas, vamos a usar una función `weekdays()` y agregaremos una columna
de dos clases, si la fecha es fin de semana se etiqueta con weekend y si
no con weekdays.

    for (i in 1:17568){
      if (weekdays(activity$date[i])=='sabado' | weekdays(activity$date[i])=='domingo'){
        activity$week[i]<-'weekend'
      }
      else{
        activity$week[i]<-'weekdays'
      }
    }

    head(activity)

    # A tibble: 6 x 4
      steps date       interval week    
      <dbl> <date>        <dbl> <chr>   
    1     0 2012-10-01        0 weekdays
    2     0 2012-10-01        5 weekdays
    3     0 2012-10-01       10 weekdays
    4     0 2012-10-01       15 weekdays
    5     0 2012-10-01       20 weekdays
    6     0 2012-10-01       25 weekdays

1.  Make a panel plot containing a time series plot (type = “l”) of the
    5-minute interval (x-axis) and the average number of steps taken,
    averaged across all weekday days or weekend days (y-axis).

En esta parte calcularemos el promedio de pasos por fin de semana y las
semanas laborales de acuerdo a los intervalos.

    filas<-dim(activity)[1]

    #Weekdays
    prom_weekdays<-c()
    for (i in 1:288){
      z<-c()
      for (j in seq(i,filas,by=288)){
        if (activity$week[j]=='weekdays'){
            z<-c(z,activity$steps[j])
        }
      }
      prom_weekdays<-c(prom_weekdays,mean(z,na.rm = TRUE))
    }
    head(prom_weekdays)

    [1] 1.7169811 0.3396226 0.1320755 0.1509434 0.0754717 1.1132075

    #Weekend

    prom_weekend<-c()
    for (i in 1:288){
      z<-c()
      for (j in seq(i,filas,by=288)){
        if (activity$week[j]=='weekend'){
            z<-c(z,activity$steps[j])
        }
      }
      prom_weekend<-c(prom_weekend,mean(z,na.rm = TRUE))
    }

    head(prom_weekend)

    [1] 0.0 0.0 0.0 0.0 0.0 6.5

Creamos los datos con la información de arriba.

    datos1<-data.frame(cbind(activity$interval[1:288],prom_weekdays))
    colnames(datos1)<-c('interval','weekdays')
    head(datos1)

      interval  weekdays
    1        0 1.7169811
    2        5 0.3396226
    3       10 0.1320755
    4       15 0.1509434
    5       20 0.0754717
    6       25 1.1132075

    datos2<-data.frame(cbind(activity$interval[1:288],prom_weekend))
    colnames(datos2)<-c('interval','weekend')
    head(datos2)

      interval weekend
    1        0     0.0
    2        5     0.0
    3       10     0.0
    4       15     0.0
    5       20     0.0
    6       25     6.5

Para graficar ambas graficas usaremos la libreria `gridExtra` y la
función `grid.arrange()`

    library(ggplot2)
    library(gridExtra)

    weekday<-ggplot(data=datos1,aes(datos1$interval,datos1$weekdays))+geom_line()+ylab('average steps for weekdays')+ xlab('Intervals 5 min')

    weekend<-ggplot(data=datos1,aes(datos1$interval,datos1$weekdays))+geom_line()+ylab('average steps for weekdays')+ xlab('Intervals 5 min')

    grid.arrange(weekday, weekend, ncol = 2, widths = c(6, 6))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-16-1.png)
