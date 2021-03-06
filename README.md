# package-info maven plugin
When you use JaxB2 to generate sources (xjc) from schema's, but the
xsd's are rather messy and you don't have ownership over them. This can 
result in xml containing multiple namespace declarations using standard 
prefixes like ns1, ns2 etc for the same namespace.

The com.sun.xml.bind.v2.runtime.JAXBContextImpl, which is responsible 
for marshalling to xml, uses a package-info.class to prefix namespaces.
This is a standard package level annotation file where namespace prefixes
can be defined.

The package-info.java should look something like:

    /** Generated by package-info-maven-plugin */
    @javax.xml.bind.annotation.XmlSchema(
        elementFormDefault = javax.xml.bind.annotation.XmlNsForm.QUALIFIED,
        xmlns = {
            @XmlNs(prefix = "", namespaceURI = "urn:foo:bar"),
            @XmlNs(prefix = "son", namespaceURI = "urn:some:other:ns")
        }
    )
    package use.this.package;
    import javax.xml.bind.annotation.XmlNs;

The standard maven jaxb2 plugins can't generate the @XmlNs annotations.

# Usage

First of all, make sure the jaxb2 xjc plugin doesn't generate the 
package-info.java anymore. This can be done by doing:

When using *org.codehaus.mojo:jaxb2-maven-plugin*:
    
    <noPackageLevelAnnotations>true</noPackageLevelAnnotations>
    
When using *org.jvnet.jaxb2.maven2:maven-jaxb2-plugin*:

    <packageLevelAnnotations>false</packageLevelAnnotations>

Then setup the new plugin like this:

    <plugin>
        <groupId>sedus.maven</groupId>
        <artifactId>package-info-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
            <usePackage>use.this.package</usePackage>
            <prefixes>
                <nsprefix>
                    <namespace>urn:foo:bar</namespace>
                    <prefix></prefix>
                </nsprefix>
                <nsprefix>
                    <namespace>urn:some:other:ns</namespace>
                    <prefix>foo</prefix>
                </nsprefix>
            </prefixes>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>generate</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

This will automatically generate the package-info.java during the
`process-sources` phase.

It needs a `usePackage`, this will be used to write the file to the 
correct directory.

Furthermore the namespaces and their prefixes need to be defined. The
configuration is quite forward so see the example.
