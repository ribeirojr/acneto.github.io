---
layout: post
title:  "Simplificando a utilização do JDBC com Spring"
date:   2014-03-29 16:00=
categories: java spring jdbc template
---

<center>*É muito comum aplicações Java utilizarem JDBC, mas infelizmente essa API exige que o desenvolvedor escreva muito boilerplate só para atender as exigências do compilador. Vou mostrar como a vida pode ser mais fácil usando Spring.*</center>

#### Introdução ao Spring

Atualmente o framework Spring é utilizado largamente no desenvolvimento de software na plataforma Java. O Spring é um framework leve, não intrusivo e que promove o uso das boas práticas de programação, como o baixo acoplamento entre componentes de software através de [Injeção de Dependência.][injecao-dependencia]

Em Java, quando precisamos realizar a persistência de dados geralmente utilizamos alguma ferramenta de mapeamento objeto-relacional, como o Hibernate para simplificar nossa vida. Porém, como se sabe, em algumas situações o SQL é uma escolha mais apropriada (por exemplo, se estivermos buscando um melhor desempenho em uma consulta). Hoje o desenvolvedor Java ao trabalhar com SQL puro precisa utilizar a famosa API JDBC. Infelizmente, essa API obriga o desenvolvedor a escrever uma quantidade enorme de código para realizar coisas relativamente simples. Para resolver esse problema, uma solução é utilizar o suporte de acesso a dados que é fornecido pelo Spring.

#### Acesso a dados com Spring

O acesso a dados no Spring é baseado em um padrão de projeto chamado `Template Method`. Esse padrão define um algoritmo base e cada subclasse pode redefini-lo com um comportamento específico. A idéia básica é que o Spring sabe que alguns passos sempre são repetidos na utilização do JDBC, como por exemplo a abertura de conexão com o banco de dados e a limpeza dos recursos alocados após o uso. Dessa forma, esses procedimentos se tornaram passos fixos no algoritmo do `Template Method`, enquanto a parte flexível do algoritmo são os métodos escritos pelo desenvolvedor.

Para manter o post curto irei apresentar apenas o `JdbcTemplate` - até porque os outros templates seguem o mesmo conceito.

#### JdbcTemplate

Vamos imaginar que temos uma classe DAO que precisa cadastrar no banco um objeto Pessoa. Vamos escrever o código JDBC da maneira convencional (listagem 1) e em seguida refatorar esse código.

{% highlight java %}
// listagem 1
public class PessoaDAOImpl implements GenericDAO<Pessoa> {

public void cadastrar(Pessoa pessoa) {
String PESSOA_INSERT = "insert into pessoa (id, nome, idade, sexo) "+ "values (null, ?,?,?)";
Connection conn = null;
PreparedStatement stmt = null;

        try {
            conn = criarConexao();
            stmt = conn.prepareStatement(PESSOA_INSERT);
            stmt.setString(1, pessoa.getNome());
            stmt.setInt(2, pessoa.getIdade());
            stmt.setString(3, String.valueOf(pessoa.getSexo()) );
            stmt.execute();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (stmt != null) {
                    stmt.close();
                }
                if (conn != null) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
{% endhighlight %}

Podemos ver que a maior parte do código escrito é apenas o código padrão exigido pela API JDBC, enquanto o que realmente nós interessa fica no meio de um amontoado de código. Agora imagine que fosse preciso escrever uma operação de `update` dentro desse DAO. Se pararmos pra pensar um pouco, vamos perceber que todo o código de update seria idêntico ao do `insert`, mudando apenas a instrução SQL, gerando um problema de duplicação de código. 

Vamos usar o Spring para simplificar as coisas, refatorando para a listagem 2.

{% highlight java %}
// listagem 2, código refatorado

public class PessoaDAOImpl implements GenericDAO<Pessoa> {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    public void cadastrar(Pessoa pessoa) {
        String PESSOA_INSERT = "insert into pessoa (id, nome, idade, sexo) " + 
        "values (null, ?,?,?)";

        jdbcTemplate.update(PESSOA_INSERT, new Object[] {
            pessoa.getNome(), pessoa.getIdade(), pessoa.getSexo()});
    }
{% endhighlight %}

O Spring exige um arquivo chamado `applicationContext.xml` (listagem 3). O aquivo `applicationContext.xml` está fazendo três coisas: 

1. configurando um `Data Source` para o banco MySQL; 
2. associando o `jdbcTemplate` ao Data Source criado;
3. injetando o `jdbcTemplate` em PessoaDAOImpl;

{% highlight xml %}
<!--listagem 3-->
<?xml version="1.0" encoding="UTF-8"?>
...
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver" />
              <property name="url" value="jdbc:mysql://localhost:3306/teste" />
              <property name="username" value="root" />
            <property name="password" value="root" />
    </bean>
    <bean id="jdbcTemplate"
        class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <bean id="pessoaDAO" class="jm.jdbctemplate.dao.PessoaDAOImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>
</beans>
{% endhighlight %}

Note que não precisamos instanciar a propriedade `jdbcTemplate diretamente dentro do DAO, o Spring faz isso de forma automática. Para realizar um insert no banco podemos fazer algo parecido com a listagem 4.

{% highlight java %}
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext(
"jm/main/jdbctemplate/applicationContext.xml");

        GenericDAO<Pessoa> pessoaDAO = (GenericDAO) ctx.getBean("pessoaDAO");
        Pessoa p = new Pessoa();
        p.setNome("carlos"); p.setIdade(20); p.setSexo("M");
        pessoaDAO.cadastrar(p);
    }
}
{% endhighlight %}

Analisando o DAO após a refatoração, a primeira coisa que podemos perceber é que a classe ficou muito menor e mais legível do que a primeira versão, o nosso novo método de cadastro agora possui apenas 2 linhas!

Agora que já estamos inserindo no banco, vamos analisar como é o procedimento para realizarmos buscas. Imagine que preciamos criar um método para localizar uma pessoa pela chave primária. Assim como antes, vamos começar apresentando o código Java da maneira convencional (listagem 5) para em seguida refatorá-lo.

{% highlight java %}
//listagem 5

public class PessoaDAOImpl implements GenericDAO<Pessoa> {

public Pessoa findByPK(int id) {

String FIND_PESSOA = "select id, nome, idade, sexo from pessoa where id = ?";
Connection conn = null;
PreparedStatement stmt = null;
ResultSet rs = null;

try {
    conn = criarConexao();
    stmt = conn.prepareStatement(FIND_PESSOA);
    stmt.setInt(1, id);
    rs = stmt.executeQuery();

    Pessoa pessoa = null;
    if (rs.next()) {
        pessoa = new Pessoa();
        pessoa.setId(rs.getInt("id"));
        pessoa.setNome(rs.getString("nome"));
        pessoa.setIdade(rs.getInt("idade"));
        pessoa.setSexo("sexo");
    }

    return pessoa;

    } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (stmt != null) {
                    stmt.close();
                }
                if (conn != null) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    return null;
}


{% endhighlight %}

Como já vimos antes, a abordagem tradicional é muito trabalhosa. Vamos usar o método `query` fornecido pelo `jdbcTemplate`.

{% highlight java %}
// listagem 6, código refatorado

public class PessoaDAOImpl implements GenericDAO<Pessoa> {

String FIND_PESSOA = "select id, nome, idade, sexo from pessoa where id = ?";

private JdbcTemplate jdbcTemplate;

public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
}

public Pessoa findByPK(int id) {

List lista = jdbcTemplate.query(FIND_PESSOA, new Object[] { Integer.valueOf(id) }, 
    new RowMapper() {

public Object mapRow(ResultSet rs, int row) throws SQLException {
                Pessoa pessoa = new Pessoa();
                pessoa.setId(rs.getInt("id"));
                pessoa.setNome(rs.getString("nome"));
                pessoa.setIdade(rs.getInt("idade"));
                pessoa.setSexo("sexo");
                return pessoa;
            }
        });
        return (Pessoa) lista.get(0);
}
{% endhighlight %}

O método query aceita três parâmetros:

1. a String SQL; 
2. um array do tipo Object que vai ser passado como parâmetro para a consulta (observe que os parâmetros são baseados em  índice, por isso a ordem deles é importante);
3. um objeto RowMapper que é o responsável por extrair os dados do ResultSet e converter para o objeto que estamos trabalhando (Pessoa). Para cada registro localizado no banco o método mapRow é chamado. Como estamos localizando pela chave primária o mapRow só será invocado uma vez.

#### Conclusão

Bem, é isso. Usar o Spring simplifica e acelera o uso do JDBC porque evita termos que escrever sempre o mesmo código repetitivo. [Recomendo a leitura da documentação ofical][spring-data-access].

[spring-data-access]:http://docs.spring.io/spring/docs/current/spring-framework-reference/html/jdbc.html
[injecao-dependencia]:http://martinfowler.com/articles/injection.html
