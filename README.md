<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Panel de Monitoreo de Fraudes</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      background-color: #f4f6f9;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    .header {
      background-color: #007bff;
      color: white;
      padding: 20px;
      text-align: center;
      border-bottom: 5px solid #0056b3;
    }

    .header h1 {
      font-size: 28px;
      font-weight: bold;
      margin-bottom: 5px;
    }

    .header p {
      margin: 0;
      font-size: 16px;
    }

    .table thead th {
      background-color: #007bff;
      color: white;
      text-align: center;
    }

    .chart-card {
      background-color: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
      margin-bottom: 30px;
    }

    .table-container {
      margin-top: 20px;
    }

    canvas {
      max-height: 350px;
    }

    .section-title {
      margin-top: 30px;
      margin-bottom: 15px;
      font-weight: bold;
      font-size: 20px;
      color: #343a40;
    }
  </style>
</head>
<body>
  <!-- Cabecera del dashboard -->
  <div class="header">
    <h1>Panel de Monitoreo de Fraudes</h1>
    <p>Trabajo de Fin de Grado - Daniel Albarracín Morales - Junio 2025</p>
  </div>

  <div class="container mt-4">
    <!-- Gráfico fijo arriba -->
    <div class="chart-card">
      <h5 class="text-center mb-4">Cantidad de fraudes por categoría</h5>
      <canvas id="categoryChart"></canvas>
    </div>

    <!-- Tabla de fraudes -->
    <div class="section-title">Historial de transacciones sospechosas</div>
    <div class="table-responsive table-container">
      <table class="table table-bordered table-hover">
        <thead>
        <tr>
          <th>Número de tarjeta</th>
          <th>Fecha de transacción</th>
          <th>ID de transacción</th>
          <th>Categoría</th>
          <th>Comercio</th>
          <th>Monto (€)</th>
          <th>Distancia</th>
          <th>Edad</th>
        </tr>
        </thead>
        <tbody id="fraud_table_body">
        </tbody>
      </table>
    </div>
  </div>

  <!-- Scripts -->
  <script src="js/jquery-1.12.4.min.js"></script>
  <script src="js/sockjs-1.1.1.min.js"></script>
  <script src="js/stomp.min.js"></script>

  <script>
    // Inicializar gráfico
    const ctx = document.getElementById('categoryChart').getContext('2d');
    const categoryChart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: [],
        datasets: [{
          label: 'Cantidad de fraudes',
          data: [],
          backgroundColor: 'rgba(0, 123, 255, 0.6)',
          borderColor: 'rgba(0, 123, 255, 1)',
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Número de fraudes'
            }
          }
        }
      }
    });

    // WebSocket
    const socket = new SockJS('/stomp');
    const stompClient = Stomp.over(socket);

    stompClient.connect({}, function () {
      stompClient.subscribe("/topic/fraudData", function (data) {
        const response = JSON.parse(data.body);
        const fraudes = response.fraudAlert;

        let filas = '';
        const contadorCategorias = {};

        fraudes.forEach(fraude => {
          filas += `
            <tr>
              <td>${fraude.cc_num}</td>
              <td>${fraude.trans_time}</td>
              <td>${fraude.trans_num}</td>
              <td>${fraude.category}</td>
              <td>${fraude.merchant}</td>
              <td>${fraude.amt}</td>
              <td>${fraude.distance}</td>
              <td>${fraude.age}</td>
            </tr>
          `;

          contadorCategorias[fraude.category] = (contadorCategorias[fraude.category] || 0) + 1;
        });

        $("#fraud_table_body").html(filas);

        // Actualizar gráfico
        categoryChart.data.labels = Object.keys(contadorCategorias);
        categoryChart.data.datasets[0].data = Object.values(contadorCategorias);
        categoryChart.update();
      });
    });
  </script>
</body>
</html>



---POM XML ---



<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.datamantra</groupId>
  <artifactId>FraudDetection</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <spark.version>2.2.1</spark.version>
    <scala.tools.version>2.11</scala.tools.version>
    <scala.version>2.11.8</scala.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-core_${scala.tools.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>

    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-sql_${scala.tools.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>

    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-mllib_${scala.tools.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>

    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-sql-kafka-0-10_2.11</artifactId>
      <version>2.2.0</version>
    </dependency>

    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
      <version>2.2.1</version>
    </dependency>

    <dependency>
      <groupId>com.databricks</groupId>
      <artifactId>spark-csv_${scala.tools.version}</artifactId>
      <version>1.5.0</version>
    </dependency>

    <dependency>
      <groupId>com.datastax.spark</groupId>
      <artifactId>spark-cassandra-connector_2.11</artifactId>
      <version>2.0.7</version>
    </dependency>

    <dependency>
      <groupId>com.datastax.cassandra</groupId>
      <artifactId>cassandra-driver-core</artifactId>
      <version>3.3.2</version>
    </dependency>

    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>2.7.2</version>
    </dependency>

    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>0.10.0.1</version>
    </dependency>

    <dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.3.3</version>
    </dependency>

    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>

    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <version>${scala.version}</version>
    </dependency>

    <dependency>
      <groupId>org.scalatest</groupId>
      <artifactId>scalatest_${scala.tools.version}</artifactId>
      <version>2.2.5</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.4</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <testSourceDirectory>src/test/scala</testSourceDirectory>
    <finalName>frauddetection-spark</finalName>

    <plugins>
      <!-- Plugin moderno para compilar Scala -->
      <plugin>
        <groupId>net.alchim31.maven</groupId>
        <artifactId>scala-maven-plugin</artifactId>
        <version>4.5.6</version>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>

      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.4</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>


----- POOOM 2 ----


<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.datamantra</groupId>
  <artifactId>creditcard-tr-producer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <scala.tools.version>2.11</scala.tools.version>
    <scala.version>2.11.8</scala.version>
    <confluent.version>3.3.1</confluent.version>
  </properties>

  <repositories>
    <repository>
      <id>confluent</id>
      <url>http://packages.confluent.io/maven/</url>
    </repository>
  </repositories>

  <dependencies>

    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>1.1.0</version>
    </dependency>

    <dependency>
      <groupId>io.confluent</groupId>
      <artifactId>kafka-avro-serializer</artifactId>
      <version>${confluent.version}</version>
    </dependency>

    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-csv</artifactId>
      <version>1.1</version>
    </dependency>

    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.2</version>
    </dependency>

    <dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.3.3</version>
    </dependency>

    <dependency>
      <groupId>com.twitter</groupId>
      <artifactId>algebird-core_${scala.tools.version}</artifactId>
      <version>0.12.0</version>
    </dependency>

    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>

    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <version>${scala.version}</version>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.4</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.scala-tools.testing</groupId>
      <artifactId>specs</artifactId>
      <version>1.6.2.2_1.5.0</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.scalatest</groupId>
      <artifactId>scalatest_${scala.tools.version}</artifactId>
      <version>2.2.5</version>
    </dependency>

  </dependencies>

  <build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <plugins>

      <!-- Plugin moderno para Scala -->
      <plugin>
        <groupId>net.alchim31.maven</groupId>
        <artifactId>scala-maven-plugin</artifactId>
        <version>4.5.6</version>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>

      <plugin>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro-maven-plugin</artifactId>
        <version>1.7.6</version>
        <executions>
          <execution>
            <id>schemas</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>schema</goal>
              <goal>protocol</goal>
              <goal>idl-protocol</goal>
            </goals>
            <configuration>
              <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>

      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>2.5.5</version>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
        </configuration>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

    </plugins>
  </build>

</project>

