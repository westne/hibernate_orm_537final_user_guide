许多组织围绕数据库对象（表、列、外键等）的命名定义规则。PhysicalNamingStrategy的思想是帮助实现这样的命名规则，而不必通过显式名称将它们硬编码到映射中。

ImplicitNamingStrategy的目的是在没有明确指定时确定名为`accountNumber`的属性映射到`accountNumber`的逻辑列名，而PhysicalNamingStrategy的目的是，例如，说明物理列名应该缩写为`acct_num`。

> 的确，在本例中，`acct_num`的解决可以在ImplicitNamingStrategy中进行处理。但关键是关注点的分离。无论属性是否显式地指定了列名，或者我们是否隐式地确定了列名，都将应用PhysicalNamingStrategy。ImplicitNamingStrategy只有在没有给出显式名称时才会应用。所以这取决于需要和意图。

默认实现是简单地使用逻辑名称作为物理名称。然而，应用程序和集成可以定义这个PhysicalNamingStrategy约定的自定义实现。下面是一个名为Acme Corp的虚拟公司的PhysicalNamingStrategy示例，其命名标准是：

* 宁愿使用下划线分隔的单词，而不要驼峰形

* 用标准缩略语替换某些单词

###### 示例2.示例PhysicalNamingStrategy实现

```java
/*
 * Hibernate, Relational Persistence for Idiomatic Java
 *
 * License: GNU Lesser General Public License (LGPL), version 2.1 or later.
 * See the lgpl.txt file in the root directory or <http://www.gnu.org/licenses/lgpl-2.1.html>.
 */
package org.hibernate.userguide.naming;

import java.util.LinkedList;
import java.util.List;
import java.util.Locale;
import java.util.Map;
import java.util.TreeMap;

import org.hibernate.boot.model.naming.Identifier;
import org.hibernate.boot.model.naming.PhysicalNamingStrategy;
import org.hibernate.engine.jdbc.env.spi.JdbcEnvironment;

import org.apache.commons.lang3.StringUtils;

/**
 * An example PhysicalNamingStrategy that implements database object naming standards
 * for our fictitious company Acme Corp.
 * <p/>
 * In general Acme Corp prefers underscore-delimited words rather than camel casing.
 * <p/>
 * Additionally standards call for the replacement of certain words with abbreviations.
 *
 * @author Steve Ebersole
 */
public class AcmeCorpPhysicalNamingStrategy implements PhysicalNamingStrategy {
	private static final Map<String,String> ABBREVIATIONS = buildAbbreviationMap();

	@Override
	public Identifier toPhysicalCatalogName(Identifier name, JdbcEnvironment jdbcEnvironment) {
		// Acme naming standards do not apply to catalog names
		return name;
	}

	@Override
	public Identifier toPhysicalSchemaName(Identifier name, JdbcEnvironment jdbcEnvironment) {
		// Acme naming standards do not apply to schema names
		return name;
	}

	@Override
	public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment jdbcEnvironment) {
		final List<String> parts = splitAndReplace( name.getText() );
		return jdbcEnvironment.getIdentifierHelper().toIdentifier(
				join( parts ),
				name.isQuoted()
		);
	}

	@Override
	public Identifier toPhysicalSequenceName(Identifier name, JdbcEnvironment jdbcEnvironment) {
		final LinkedList<String> parts = splitAndReplace( name.getText() );
		// Acme Corp says all sequences should end with _seq
		if ( !"seq".equalsIgnoreCase( parts.getLast() ) ) {
			parts.add( "seq" );
		}
		return jdbcEnvironment.getIdentifierHelper().toIdentifier(
				join( parts ),
				name.isQuoted()
		);
	}

	@Override
	public Identifier toPhysicalColumnName(Identifier name, JdbcEnvironment jdbcEnvironment) {
		final List<String> parts = splitAndReplace( name.getText() );
		return jdbcEnvironment.getIdentifierHelper().toIdentifier(
				join( parts ),
				name.isQuoted()
		);
	}

	private static Map<String, String> buildAbbreviationMap() {
		TreeMap<String,String> abbreviationMap = new TreeMap<> ( String.CASE_INSENSITIVE_ORDER );
		abbreviationMap.put( "account", "acct" );
		abbreviationMap.put( "number", "num" );
		return abbreviationMap;
	}

	private LinkedList<String> splitAndReplace(String name) {
		LinkedList<String> result = new LinkedList<>();
		for ( String part : StringUtils.splitByCharacterTypeCamelCase( name ) ) {
			if ( part == null || part.trim().isEmpty() ) {
				// skip null and space
				continue;
			}
			part = applyAbbreviationReplacement( part );
			result.add( part.toLowerCase( Locale.ROOT ) );
		}
		return result;
	}

	private String applyAbbreviationReplacement(String word) {
		if ( ABBREVIATIONS.containsKey( word ) ) {
			return ABBREVIATIONS.get( word );
		}

		return word;
	}

	private String join(List<String> parts) {
		boolean firstPass = true;
		String separator = "";
		StringBuilder joined = new StringBuilder();
		for ( String part : parts ) {
			joined.append( separator ).append( part );
			if ( firstPass ) {
				firstPass = false;
				separator = "_";
			}
		}
		return joined.toString();
	}
}
```

有多种方法可以指定要使用的PhysicalNamingStrategy。首先，应用程序可以使用hibernate.physical\_naming\_strategy配置设置指定实现，该设置接受：

* 引用实现`org.hibernate.boot.model.naming.PhysicalNamingStrategy`约定的类

* 实现`org.hibernate.boot.model.naming.PhysicalNamingStrategy`约定的类的FQN

其次，应用程序和集成可以利用`org.hibernate.boot.MetadataBuilder#applyPhysicalNamingStrategy`。有关引导的其他详细信息，请参阅引导程序。



