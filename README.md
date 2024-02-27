import ldap3

class LDAPClient:
    def __init__(self, server, port, bind_dn, bind_password):
        self.server = server
        self.port = port
        self.bind_dn = bind_dn
        self.bind_password = bind_password
        self.connection = None

    def connect(self):
        self.connection = ldap3.Connection(
            ldap3.Server(self.server, port=self.port),
            user=self.bind_dn,
            password=self.bind_password,
            auto_bind=True
        )

    def search(self, base_dn, search_filter):
        self.connection.search(
            search_base=base_dn,
            search_filter=search_filter,
            attributes=ldap3.ALL_ATTRIBUTES
        )
        return self.connection.entries

    def add(self, dn, attributes):
        self.connection.add(dn, attributes)

    def modify(self, dn, changes):
        self.connection.modify(dn, changes)

    def delete(self, dn):
        self.connection.delete(dn)

    def disconnect(self):
        if self.connection:
            self.connection.unbind()

# Example usage
if __name__ == "__main__":
    ldap_client = LDAPClient("ldap.example.com", 389, "cn=admin,dc=example,dc=com", "admin_password")
    ldap_client.connect()

    # Search for entries
    search_results = ldap_client.search("dc=example,dc=com", "(objectclass=person)")
    for entry in search_results:
        print("Entry:", entry.entry_dn, entry.entry_attributes_as_dict)

    # Add an entry
    new_entry = {
        "cn": "John Doe",
        "sn": "Doe",
        "mail": "john@example.com",
        "objectClass": ["person", "inetOrgPerson"]
    }
    ldap_client.add("cn=John Doe,dc=example,dc=com", new_entry)

    # Modify an entry
    modify_changes = {'mail': [(ldap3.MODIFY_REPLACE, ['new_email@example.com'])]}
    ldap_client.modify("cn=John Doe,dc=example,dc=com", modify_changes)

    # Delete an entry
    ldap_client.delete("cn=John Doe,dc=example,dc=com")

    ldap_client.disconnect()
